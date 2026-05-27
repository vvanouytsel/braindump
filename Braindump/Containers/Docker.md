---
tags:
  - containers
---

## Working with Registries

- Show images in registry

```bash
❯ curl localhost:30050/v2/_catalog

{"repositories":["bitnami/kubectl","bitnami/rabbitmq-cluster-operator","bitnami/rmq-messaging-topology-operator","external-secrets/external-secrets","fluent/fluent-bit","ingress-nginx/controller","ingress-nginx/kube-webhook-certgen","jimmidyson/configmap-reload","kube-state-metrics/kube-state-metrics","kubebuilder/kube-rbac-proxy","percona/percona-postgresql-operator","percona/pmm-client","prom/pushgateway","prometheus/alertmanager","prometheus/node-exporter","prometheus/prometheus","prometheus-adapter/prometheus-adapter","prometheuscommunity/postgres-exporter","my-api-console","my-assets","my-compute","my-config-hub","my-config-operator","my-context","my-context-hub","my-context-sql-sync","my-custom-calculations-runtime","my-dash-hub","my-dashboarding","my-datasource","my-error-pages","my-filters","my-fingerprints","my-hps","my-keycloak","my-keycloak-config","my-migrate-hps-postgres","my-ml-enterprise-gateway","my-ml-enterprise-gateway-kernel-python-311","my-ml-enterprise-gateway-kernel-python-viz","my-ml-enterprise-gateway-kernelspecs","my-ml-hub","my-ml-jupyter-server","my-notifications","my-postgres","my-postgres-configuration","my-rabbitmq","my-timeseries-builder","my-translations","my-trend-hub2","my-work-organisation","my-zementis","velero/velero","velero/velero-plugin-for-aws","velero/velero-plugin-for-microsoft-azure"]}
```

- Show specific tags of an image

```bash
# curl localhost:30050/v2/$IMAGE/tags/list
❯ curl localhost:30050/v2/bitnami/kubectl/tags/list

{"name":"bitnami/kubectl","tags":["1.28"]}
```


## Do not use build arguments for secrets

You do not want to store any secrets in your container. This is why you do not use the `ENV` keyword in your Dockerfile for secrets, as these will be persisted in your image.

Instead, I used the `ARG` keyword. These are available during the build context but are not persisted in your final image. However recently it came to my attention that this is also a very bad practice.

Even though your `ARG` is not visible as an environment variable in your final image. Everyone that has download your image can inspect it and find out whatever you specified in your `ARG` during building.

Assume you have the following Dockerfile:
```
FROM ubuntu:latest
ARG my-secret
```

Typically you would build pass the secret during build time so you do not have to store it in git. Because it is a `ARG`, it will also not be visible as an environment variable in the final image.

```
$ docker build -t build-args:1.0.0 --build-arg my-secret=somesecretpassword .
```


However, here is the danger. You can still inspect the layers and you can see the value of the secret. One of the commands to do so is `docker history`.

```
$ docker history build-args:1.0.0                                             
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
67b8d477c595   8 months ago   ARG my-secret=somesecretpassword                0B        buildkit.dockerfile.v0
<missing>      8 months ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      8 months ago   /bin/sh -c #(nop) ADD file:6df775300d76441aa…   78.1MB    
<missing>      8 months ago   /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      8 months ago   /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      8 months ago   /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B        
<missing>      8 months ago   /bin/sh -c #(nop)  ARG RELEASE                  0B 
```

Never use `ARG` or `ENV` in your Dockerfile for secrets. Instead you have to use [build secrets](https://docs.docker.com/build/building/secrets/).
You can use the `--secret` flag to pass a secret. 

Assume the following Dockerfile:

```
FROM ubuntu:latest
RUN --mount=type=secret,id=my-secret \
cat /run/secrets/my-secret >> /root/my-secret.txt
```

A secret has an id and can either be an environment variable or a file. In this case we define it as an environment variable `MY_SECRET` and set the id to `my_secret`. Docker mounts the secret in your container in `/run/secrets/$your_secret_id`.

```
export MY_SECRET=myverysecretpassword ; docker build --secret id=my-secret,env=MY_SECRET -t build-args:1.0.0 .
```

You can see that the value of our secret is not visible via the `docker history` command.

```
$ docker history build-args:1.0.0                                                                               
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
60a8f16082c3   15 seconds ago   RUN /bin/sh -c cat /run/secrets/my-secret >>…   20B       buildkit.dockerfile.v0
<missing>      8 months ago     /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      8 months ago     /bin/sh -c #(nop) ADD file:6df775300d76441aa…   78.1MB    
<missing>      8 months ago     /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      8 months ago     /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B        
<missing>      8 months ago     /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B        
<missing>      8 months ago     /bin/sh -c #(nop)  ARG RELEASE                  0B  
```

The result is that the secret was available for building your container, but no traces of it were left behind.

```
$ docker run -ti build-args:1.0.0 cat /root/my-secret.txt
myverysecretpassword
```


My default your secret is mounted to a file. You can also specify the `env` option to have it set as an environment during your build.
```
RUN --mount=type=secret,id=my-secret,env=MY_SECRET_ENV_VAR
```

## Layers

Assume that you have the following Dockerfile.

```Dockerfile
FROM ubuntu:24.04
COPY myfile.txt /myfile.txt
RUN apt-get update && apt-get install -y vim
```


If you build that, you will see that it contains 3 layers. One for each instruction in your Dockerfile.


```bash

❯ docker build -t layerdeepdive:1.0.0 .
[+] Building 8.0s (8/8) FINISHED docker:default
=> [internal] load build definition from Dockerfile 0.0s
=> => transferring dockerfile: 188B 0.0s
=> [internal] load metadata for docker.io/library/ubuntu:24.04 0.4s
=> [internal] load .dockerignore 0.0s
=> => transferring context: 2B 0.0s
=> [internal] load build context 0.1s
=> => transferring context: 117B 0.0s
=> [1/3] FROM docker.io/library/ubuntu:24.04@sha256:c4a8d5503dfb2a3eb8ab5f807da5bc69a85730fb49b5cfca2330194ebcc41c7b 0.0s
=> => resolve docker.io/library/ubuntu:24.04@sha256:c4a8d5503dfb2a3eb8ab5f807da5bc69a85730fb49b5cfca2330194ebcc41c7b 0.0s
=> [2/3] COPY myfile.txt /myfile.txt 0.1s
=> [3/3] RUN apt-get update && apt-get install -y vim 6.5s
=> exporting to image 0.7s
=> => exporting layers 0.7s
=> => writing image sha256:f1b85c010918489758a55fadea5c7ab7ad7017ed020709e0c044cbce232bdb24 0.0s
=> => naming to docker.io/library/layerdeepdive:1.0.0 0.0s
```

Using the `history` command you can see the build history of an image. Entries with a size greater than 0 are actual layers. Entries with a size of 0B just set metadata, these are not real filesystem layers.


```bash
❯ docker image history layerdeepdive:1.0.0
IMAGE CREATED CREATED BY SIZE COMMENT
f1b85c010918 4 minutes ago RUN /bin/sh -c apt-get update && apt-get ins… 134MB buildkit.dockerfile.v0
<missing> 4 minutes ago COPY myfile.txt /myfile.txt # buildkit 21B buildkit.dockerfile.v0
<missing> 6 weeks ago /bin/sh -c #(nop) CMD ["/bin/bash"] 0B
<missing> 6 weeks ago /bin/sh -c #(nop) ADD file:8ce1caf246e7c778b… 78.1MB
<missing> 6 weeks ago /bin/sh -c #(nop) LABEL org.opencontainers.… 0B
<missing> 6 weeks ago /bin/sh -c #(nop) ARG LAUNCHPAD_BUILD_ARCH 0B
<missing> 6 weeks ago /bin/sh -c #(nop) ARG RELEASE 0B
```

  
  

You can inspect the layers via the inspect command.

  

```bash
❯ docker inspect layerdeepdive:1.0.0
[
{
"Id": "sha256:f1b85c010918489758a55fadea5c7ab7ad7017ed020709e0c044cbce232bdb24",
"RepoTags": [
"layerdeepdive:1.0.0"
],
"RepoDigests": [],
"Comment": "buildkit.dockerfile.v0",
"Created": "2026-05-27T09:41:38.795638023+02:00",
"Config": {
"Env": [
"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
],
"Cmd": [
"/bin/bash"
],
"Labels": {
"org.opencontainers.image.version": "24.04"
}
},
"Architecture": "amd64",
"Os": "linux",
"Size": 211933402,
"GraphDriver": {
"Data": {
"LowerDir": "/var/lib/docker/overlay2/yqrrowrqzljg0j10us9z8abgh/diff:/var/lib/docker/overlay2/3d6941ad32c16ce745bca4d5f88653d8b679d66ffdf260e628bde1c46f7b029c/diff",
"MergedDir": "/var/lib/docker/overlay2/fk6yo7lvdklxrdi986fmkwjn5/merged",
"UpperDir": "/var/lib/docker/overlay2/fk6yo7lvdklxrdi986fmkwjn5/diff",
"WorkDir": "/var/lib/docker/overlay2/fk6yo7lvdklxrdi986fmkwjn5/work"
},
"Name": "overlay2"
},
"RootFS": {
"Type": "layers",
"Layers": [
"sha256:538812a4b9bd45adaac2b5e5b967daa6999aa44eb110aa32ae7c69702b906475",
"sha256:878fc66709422b296fa90b605f1809b0baf55f249f7cf4b2a6a71e5bebe82d0c",
"sha256:4a82024f524bd6b020d7158edf6d19f0651bc2cf8549263c14258ddb72cc2d4d"
]
},
"Metadata": {
"LastTagTime": "2026-05-27T09:41:39.483377205+02:00"
}
}
]
```

  
To only show the layers:
  

```bash
❯ docker inspect layerdeepdive:1.0.0 --format '{{json .RootFS.Layers}}' | jq
[
"sha256:538812a4b9bd45adaac2b5e5b967daa6999aa44eb110aa32ae7c69702b906475",
"sha256:878fc66709422b296fa90b605f1809b0baf55f249f7cf4b2a6a71e5bebe82d0c",
"sha256:4a82024f524bd6b020d7158edf6d19f0651bc2cf8549263c14258ddb72cc2d4d"
]
```

Together with the `docker image history` command we ran earlier and the `Dockerfile` we have, we can see where these layers are coming from.  

`sha256:538812a4b9bd45adaac2b5e5b967daa6999aa44eb110aa32ae7c69702b906475` comes from the `ADD` entry that was used to build the `ubuntu:24.04` base image that we are referencing. We can double check this by pulling the `ubuntu:24.04` imare ourselves and listing its layers. As you can see that layer is identical to the layer we have in our container image.
  
```bash
<missing> 6 weeks ago /bin/sh -c #(nop) ADD file:8ce1caf246e7c778b… 78.1MB
```


```bash
❯ docker pull ubuntu:24.04
24.04: Pulling from library/ubuntu
Digest: sha256:c4a8d5503dfb2a3eb8ab5f807da5bc69a85730fb49b5cfca2330194ebcc41c7b
Status: Downloaded newer image for ubuntu:24.04
docker.io/library/ubuntu:24.04

❯ docker inspect ubuntu:24.04 --format '{{json .RootFS.Layers}}' | jq
[
"sha256:538812a4b9bd45adaac2b5e5b967daa6999aa44eb110aa32ae7c69702b906475"
]
```

`sha256:878fc66709422b296fa90b605f1809b0baf55f249f7cf4b2a6a71e5bebe82d0c` comes from the `COPY myfile.txt /myfile.txt` entry in our Dockerfile.

`sha256:4a82024f524bd6b020d7158edf6d19f0651bc2cf8549263c14258ddb72cc2d4d` comes from the `RUN apt-get update && apt-get install -y vim` entry in our Dockerfile.
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
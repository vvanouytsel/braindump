---
tags:
  - containers
---

[BuildKit](https://docs.docker.com/build/buildkit/) is an improved backend which replaces the legacy builder. It has been made the standard since Docker Engine version 23.0.

# Frontend and backend

You can consider BuildKit to be the backend. Behind the scenes a [Low-Level Build (LLB)](https://github.com/moby/buildkit#exploring-llb) is used. This binary format allows developers to extend BuildKit.

In order to interact with BuildKit you can use a human readable frontend that converts the instructions to be compatible with LLB, so that BuildKit can execute it. One of those frontends is the Dockerfile frontend.

# Builders

A builder is a BuildKit daemon that is used to run builds. It takes a Dockerfile as input and prodcues a container image. You can have multiple different builders but any building action is ran on a single builder.

>[!info] Difference between `docker build` and `docker buildx build` commands.
> When using `docker build`, you reference the default builder. With `docker buildx build` you decouple the client and the default builder, allowing you to specify different builders. To keep things simple, the full `docker buildx build` command should be used instead.

* List the available builders

```bash
$ docker buildx ls

NAME/NODE     DRIVER/ENDPOINT   STATUS    BUILDKIT   PLATFORMS
default*      docker                                 
 \_ default    \_ default       running   v0.18.2    linux/amd64 (+3), linux/arm64, linux/arm (+2), linux/ppc64le, (7 more)

```

* Switch to a different builder

```bash
$ docker buildx use <name>
```

## Drivers

Configuration of your builders is done via drivers. There are four drivers in total.

| Driver                                                       | Description                                         |
| ------------------------------------------------------------ | --------------------------------------------------- |
| [[BuildKit#Docker driver\|docker]]                           | Default, included in docker daemon                  |
| [[BuildKit#Docker container build driver\|docker-container]] | BuildKit runs as a docker container<br>             |
| [[BuildKit#Kubernetes driver\|kubernetes]]                   | BuildKit runs as pods in Kubernetes                 |
| [[BuildKit#Remote driver\|remote]]                           | Connect to a BuildKit daemon that is already set up |

| Feature                  | docker         | docker-container | kubernetes | remote             |
| ------------------------ | -------------- | ---------------- | ---------- | ------------------ |
| Automatically load image | ✅              |                  |            |                    |
| Cache export             | ✅(but not all) | ✅                | ✅          | ✅                  |
| Tarball output           |                | ✅                | ✅          | ✅                  |
| Multi-arch images        |                | ✅                | ✅          | ✅                  |
| BuildKit configuration   |                | ✅                | ✅          | Managed externally |

### Importing images in the local image store

When using the default `docker` driver, build images are automatically imported in the local image store. This is not the case when you use a builder with a different driver. The image is only exported to the build cache. If you want to load an image in the local image store, you can do so using the `--load` flag.

```bash
$ docker buildx build --load -t <image> --builder=container .
```

You can also configure the driver to automatically load created images into the local image store by setting the `default-load` option.

```bash
$ docker buildx create --driver-opt default-load=true
```

### [Docker driver](https://docs.docker.com/build/builders/drivers/docker/)

This is the default driver that is used. It is integrated directly in the Docker Engine and cannot be configured.

### [Kubernetes driver](https://docs.docker.com/build/builders/drivers/kubernetes/)

This driver allows you to connect to builders that are running in Kubernetes clusters.

```bash
 $ docker buildx create \
  --bootstrap \
  --name=kube \
  --driver=kubernetes \
  --driver-opt=[key=value,...]
```

The BuildKit will be deployed as a pod in your Kubernetes cluster. When the builder is removed, the pods are also deleted.

```bash
$ docker buildx create --bootstrap --name vvanouytsel --driver kubernetes --driver-opt namespace=vvanouytsel
[+] Building 1.3s (1/1) FINISHED                                                                                                                                                                                                                                
 => [internal] booting buildkit                                                                                                                                                                                                                            1.3s
 => => waiting for 1 pods to be ready, timeout: 2 minutes                                                                                                                                                                                                  1.2s
vvanouytsel


$ kubectl get pods -n vvanouytsel                                                                           
NAME                            READY   STATUS    RESTARTS   AGE
vvanouytsel0-65cdf6f67f-hs6xh   1/1     Running   0          3s


$ docker buildx rm vvanouytsel   
vvanouytsel removed

$ kubectl get pods -n vvanouytsel                                                                           
No resources found in vvanouytsel namespace.
```

#### Multi-platform builds

It is possible to use QEMU, but much more interesting is to use native nodes to build multi platform images. Using the kubernetes driver we can provision a set of host running x86 architecture and a set of host running an ARM architecture. A label can be set on these nodes to indicate their architecture.

BuildKit has a parameter called `node`. A node references a set of Kubernetes nodes in this case. Meaning you can create a single BuildKit builder that has a `amd64` and a `arm64` node.

```bash
# Create a builder that runs on nodes with 'kubernetes.io/arch=amd64' label
$ docker buildx create \
  --bootstrap \
  --name=vvanouytsel-builder \
  --driver=kubernetes \
  --platform=linux/amd64 \
  --node=builder-amd64 \
  --driver-opt=namespace=vvanouytsel,nodeselector="kubernetes.io/arch=amd64"
[+] Building 1.3s (1/1) FINISHED                                                                                                                                                                                                                                
 => [internal] booting buildkit                                                                                                                                                                                                                            1.3s
 => => waiting for 1 pods to be ready, timeout: 2 minutes                                                                                                                                                                                                  1.2s
vvanouytsel-builder


# Append an arm64 node to an existing builder
$ docker buildx create \
  --append \
  --bootstrap \
  --name=vvanouytsel-builder \
  --driver=kubernetes \
  --platform=linux/arm64 \
  --node=builder-arm64 \
  --driver-opt=namespace=vvanouytsel,nodeselector="kubernetes.io/arch=arm64"


# A single builder exists with multiple nodes
$ docker buildx ls
NAME/NODE             DRIVER/ENDPOINT                                                              STATUS     BUILDKIT   PLATFORMS
kube                  kubernetes                                                                                         
 \_ builder-amd64      \_ kubernetes:///kube?deployment=builder-amd64&kubeconfig=                  inactive              linux/amd64*
vvanouytsel-builder   kubernetes                                                                                         
 \_ builder-amd64      \_ kubernetes:///vvanouytsel-builder?deployment=builder-amd64&kubeconfig=   running    v0.18.2    linux/amd64*
 \_ builder-arm64      \_ kubernetes:///vvanouytsel-builder?deployment=builder-arm64&kubeconfig=   running    v0.18.2    linux/arm64*
default*              docker                                                                                             
 \_ default            \_ default                                                                  running    v0.18.2    linux/amd64 (+3), linux/arm64, linux/arm (+2), linux/ppc64le, (7 more)
```

#### Rootless mode

By default the spawned pods have a securityContext `privileged: true`.  This gives these pods access to the host resources and kernel capabilities.

You can configure your build to run in rootless mode by passing it as a flag.

```bash
 ❯  docker buildx create \
  --bootstrap \
  --name=vvanouytsel-rootless \
  --driver=kubernetes \
  --driver-opt=namespace=vvanouytsel,rootless=true
vvanouytsel-rootless
```

You can use your custom builder by specifying it during your `buildx bulid` command.

```bash
docker buildx build --builder=vvanouytsel-rootless --load -t vvanouytsel:test .
```

### Remote driver

Clone the buildkit repository.

```bash
$ git clone https://github.com/moby/buildkit.git 
```

Create certificates.

```bash
$ sudo dnf install -y nss-tools 
$ curl -JLO "https://dl.filippo.io/mkcert/latest?for=linux/amd64"
$ chmod +x mkcert-v*-linux-amd64
$ sudo cp mkcert-v*-linux-amd64 /usr/local/bin/mkcert
$ examples/kubernetes/create-certs.sh 127.0.0.1  
$ kubectl apply -f .certs/buildkit-daemon-certs.yaml -n vvanouytsel
```

Deploy the service and deployment.

```bash
$ kubectl apply -f examples/kubernetes/deployment+service.privileged.yaml -n vvanouytsel
$ kubectl scale --replicas=10 deployment/buildkitd -n vvanouytsel
```

Create a builder that points to your builtkitd.

```bash
# Since this uses a service, this only works from within the cluster.
$ kubectl port-forward service/buildkitd 1234 -n vvanouytsel
$ docker buildx create \             
  --name remote-vvanouytsel \
  --driver remote \
  --driver-opt cacert=${PWD}/.certs/client/ca.pem,cert=${PWD}/.certs/client/cert.pem,key=${PWD}/.certs/client/key.pem,servername=127.0.0.1 \
  tcp://localhost:1234
```

Build a Dockerfile in your current directory by using the remote builder and store the image as a tarball in `x.tar`.

```bash
$ docker buildx build --builder=remote-vvanouytsel -o - -t example:test . > x.tar
```

You can also directly load the created image in your image store.

```bash
$ docker buildx build --builder=remote-vvanouytsel  --load -t example:vvanouytsel .

$ docker image ls | grep example

example                                                                               vvanouytsel       d78cd6e87bb6   2 months ago    101MB
```

# Varia

## Provenance

If you are building your container image with [provenance](https://docs.docker.com/build/metadata/attestations/slsa-provenance/) enabled, your container image will include extra metadata about the build process.

This is really cool as you can find a lot of information, such as the commit, author, branch, which Dockerfile was used and much more.

Here is an example of a `jetspotter` image that shows the link to the workflow run where it was build. Cool stuff!

```bash
$ docker buildx imagetools inspect ghcr.io/vvanouytsel/jetspotter:latest --format '{{json .Provenance}}' | grep -A 3 runDetails
      "runDetails": {
        "builder": {
          "id": "https://github.com/vvanouytsel/jetspotter/actions/runs/26227743279"
        },
```
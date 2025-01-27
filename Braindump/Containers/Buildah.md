[Buildah](https://github.com/containers/buildah) is a tool to build OCI-compatible container images. It is build from the ground up to run without a daemon and does not need root permissions.

It uses the [container-storage](https://github.com/containers/storage) and [containers-image](https://github.com/containers/image) projects under the hood. Images can be build from existing images, from scratch or by using Dockerfiles. The resulting images adhere to the OCI specification and thus can run in a Docker environment.

>[!warning]
> Buildah is supported on Linux, but not on Mac and Windows.

Although it not a requirement, Buildah and Podman are often used [together](https://github.com/containers/buildah#buildah-and-podman-relationship). Buildah is the tool that you use to build your container image and with Podman you manage the container image. For example by pulling, pushing, deleting or running it.

# Getting started

* Install buildah

```bash
$ dnf install -y buildah
```

* List all images

```bash
$ buildah images
```

* List all containers

```bash
$ buildah containers
```

* Interacting with containers

```bash
# Build from fedora base image
# The command returns the name of the container, which we can use later to run it
$ container=$(buildah from fedora)
$ buildah run $container bash

# Install java inside the container
$ buildah run $ container -- dnf install -y java
```

# The power of Buildah

With Buildah you are really in control of what exactly needs to be part of your container images. As shown in the [examples](https://github.com/containers/buildah/blob/main/docs/tutorials/01-intro.md), you can use it for fine grained control over what exaclty needs to be part of your images.

In traditional methods using Dockerfiles you execute a set of RUN commands. It is possible that some files are part of your container image that have no specific reason to be there.

With Buildah we can mount the container filesystem and use your host to copy over the exact files that need to be part of the images. We can commit our changes in a layer and we do this as much as we want. This means that we can start from scratch (literally nothing) and only add the exact tools that we need, keeping the image size extremely small.

# The downsides of Buildah

Personally, I think it looks cool on paper. But generally to smaller container image sizes do not match up to the ease of use you get by writing Dockerfiles. The fact that it does not run on Windows and Mac is also another downside.

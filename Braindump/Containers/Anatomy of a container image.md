---
tags:
  - containers
  - development
---
The goal of this post is to dissect a container image step by step to get a better understanding how container images work. It's not magic after all.

Reference: https://docs.cloud.google.com/kubernetes-engine/docs/concepts/about-container-images#structure_of_an_image

An image consists of the following components:
* An image manifest
* A configuration object
* An array of one or more file system layers
* An optional image index

## Crane

Even though the files that we will talk about are present on disk, navigating them can be hard. Have a look for yourself:
```bash
sudo ls /var/lib/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots
```

To make it easier, we can use `crane`. Make sure to [install](https://github.com/google/go-containerregistry/blob/main/cmd/crane/README.md) it first.
## Image manifests

Let's dissect the `ubuntu:22.04` container image. Pull it locally first.
```bash
docker pull ubuntu:22.04
```

An image manifest provides a configuration and set of layers for a single container image for a specific architecture and operating system.

We can fetch all the manifests of a container image using crane.
```bash
crane manifest ubuntu:22.04 | jq

{
  "manifests": [
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "amd64",
        "org.opencontainers.image.base.name": "scratch",
        "org.opencontainers.image.created": "2026-05-09T00:00:00Z",
        "org.opencontainers.image.revision": "64196f4d48be3c0a3eb429067c1a8dd995cf2308",
        "org.opencontainers.image.source": "https://git.launchpad.net/cloud-images/+oci/ubuntu-base",
        "org.opencontainers.image.url": "https://hub.docker.com/_/ubuntu",
        "org.opencontainers.image.version": "22.04"
      },
      "digest": "sha256:ce941a2a18bbb922e434d6d6d2b31e571a5c3826eaf6ada0a41dcc905bd2d906",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      },
      "size": 424
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "amd64",
        "vnd.docker.reference.digest": "sha256:ce941a2a18bbb922e434d6d6d2b31e571a5c3826eaf6ada0a41dcc905bd2d906",
        "vnd.docker.reference.type": "attestation-manifest"
      },
      "digest": "sha256:ac2e2d41a4642b047ecbee7e433ed91f838a6780cd655b21e145877e7e4987ba",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "unknown",
        "os": "unknown"
      },
      "size": 562
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "arm32v7",
        "org.opencontainers.image.base.name": "scratch",
        "org.opencontainers.image.created": "2026-05-09T00:00:00Z",
        "org.opencontainers.image.revision": "73da2f53d09a985fe9fee5158749db23434258aa",
        "org.opencontainers.image.source": "https://git.launchpad.net/cloud-images/+oci/ubuntu-base",
        "org.opencontainers.image.url": "https://hub.docker.com/_/ubuntu",
        "org.opencontainers.image.version": "22.04"
      },
      "digest": "sha256:a6b15d7fe832391e9b7c4f9188bb35512407a7290cdd844681de2bdb699c786c",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "arm",
        "os": "linux",
        "variant": "v7"
      },
      "size": 424
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "arm32v7",
        "vnd.docker.reference.digest": "sha256:a6b15d7fe832391e9b7c4f9188bb35512407a7290cdd844681de2bdb699c786c",
        "vnd.docker.reference.type": "attestation-manifest"
      },
      "digest": "sha256:be8fe43792b1c9033f416be3aa22f2e8f39d61765ae6799cecd6d56ec31fd9f4",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "unknown",
        "os": "unknown"
      },
      "size": 562
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "arm64v8",
        "org.opencontainers.image.base.name": "scratch",
        "org.opencontainers.image.created": "2026-05-09T00:00:00Z",
        "org.opencontainers.image.revision": "a9bc949635aa4286cc5b4823ef836c7f3d3cf876",
        "org.opencontainers.image.source": "https://git.launchpad.net/cloud-images/+oci/ubuntu-base",
        "org.opencontainers.image.url": "https://hub.docker.com/_/ubuntu",
        "org.opencontainers.image.version": "22.04"
      },
      "digest": "sha256:c1fc012913af7a4dd0d86553d9dae19b323e7fb60d5407e800cbfbc8f7e6aa63",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "arm64",
        "os": "linux",
        "variant": "v8"
      },
      "size": 424
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "arm64v8",
        "vnd.docker.reference.digest": "sha256:c1fc012913af7a4dd0d86553d9dae19b323e7fb60d5407e800cbfbc8f7e6aa63",
        "vnd.docker.reference.type": "attestation-manifest"
      },
      "digest": "sha256:05849b301927973a6db46197cb8b80bd8648d78976896c31e87c9eea2a2b1b6d",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "unknown",
        "os": "unknown"
      },
      "size": 562
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "ppc64le",
        "org.opencontainers.image.base.name": "scratch",
        "org.opencontainers.image.created": "2026-05-09T00:00:00Z",
        "org.opencontainers.image.revision": "dfd0164612df0ef53a84c031fd5221be120d9e36",
        "org.opencontainers.image.source": "https://git.launchpad.net/cloud-images/+oci/ubuntu-base",
        "org.opencontainers.image.url": "https://hub.docker.com/_/ubuntu",
        "org.opencontainers.image.version": "22.04"
      },
      "digest": "sha256:80435fb65985ed3da6442908da1f4e893b00ebbe77cdffb234aad08b22314330",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "ppc64le",
        "os": "linux"
      },
      "size": 424
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "ppc64le",
        "vnd.docker.reference.digest": "sha256:80435fb65985ed3da6442908da1f4e893b00ebbe77cdffb234aad08b22314330",
        "vnd.docker.reference.type": "attestation-manifest"
      },
      "digest": "sha256:5f83fc9e9e4a0cf811f8ea2de788b9623854fbde0cec41d7e4a16d94f03892cc",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "unknown",
        "os": "unknown"
      },
      "size": 562
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "riscv64",
        "org.opencontainers.image.base.name": "scratch",
        "org.opencontainers.image.created": "2026-05-09T00:00:00Z",
        "org.opencontainers.image.revision": "c15d54b5d5bb5af6a55ca89fc9dc1911cd996565",
        "org.opencontainers.image.source": "https://git.launchpad.net/cloud-images/+oci/ubuntu-base",
        "org.opencontainers.image.url": "https://hub.docker.com/_/ubuntu",
        "org.opencontainers.image.version": "22.04"
      },
      "digest": "sha256:f016f905490ce3b7ef07fd557810c65e44577798beccc6b191312072e8ff27fa",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "riscv64",
        "os": "linux"
      },
      "size": 424
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "riscv64",
        "vnd.docker.reference.digest": "sha256:f016f905490ce3b7ef07fd557810c65e44577798beccc6b191312072e8ff27fa",
        "vnd.docker.reference.type": "attestation-manifest"
      },
      "digest": "sha256:8eb9a9b57846f5de5661ee4f0850e5a19148b5cc5eabae780159c8f02abdf88c",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "unknown",
        "os": "unknown"
      },
      "size": 562
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "s390x",
        "org.opencontainers.image.base.name": "scratch",
        "org.opencontainers.image.created": "2026-05-09T00:00:00Z",
        "org.opencontainers.image.revision": "047a1f1841bbf724bfe9c23898967bc7ad377b24",
        "org.opencontainers.image.source": "https://git.launchpad.net/cloud-images/+oci/ubuntu-base",
        "org.opencontainers.image.url": "https://hub.docker.com/_/ubuntu",
        "org.opencontainers.image.version": "22.04"
      },
      "digest": "sha256:1a4c892c44d3a63d9b603f17d4fe1d1ff8813065fffc18c7baa7c623de8dfe48",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "s390x",
        "os": "linux"
      },
      "size": 424
    },
    {
      "annotations": {
        "com.docker.official-images.bashbrew.arch": "s390x",
        "vnd.docker.reference.digest": "sha256:1a4c892c44d3a63d9b603f17d4fe1d1ff8813065fffc18c7baa7c623de8dfe48",
        "vnd.docker.reference.type": "attestation-manifest"
      },
      "digest": "sha256:b73786052086eb5af8aae3ecd2667bb9a115b57e7040ac24a26cadf65b6d34cd",
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "platform": {
        "architecture": "unknown",
        "os": "unknown"
      },
      "size": 562
    }
  ],
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "schemaVersion": 2
}
```


You can list a single manifest for a specific platform.

```bash
crane manifest --platform linux/amd64 ubuntu:22.04 | jq

{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 2051,
    "digest": "sha256:86f1a8d7b38e7a014c249cf2ca573c8ff7ce3cca128c5c06dcee758813726f90"
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 29736684,
      "digest": "sha256:40d16f30db405106ef8074779bdf41f012465c2a785bbeaa2eab9f2081099b47"
    }
  ]
}
```

As you can see, a manifest containers a pointer to a configuration and to a set of layers. 
The `digest` is just a unique shasum that can be used to address that object.

In our example, our `ubuntu:22.04` image only has a single layer. Layers contain the difference with the previous layer. In our case we only have a single layer, which contains the full root filesystem.

## Layers

A container image is composed of one or multiple layers. Each layer is just a tarball that contains the changes that were made related to the previous layers. Dockerfile keywords such as `FROM`, `COPY`, `ADD` and `RUN` will create a new layer.

Let's analyze the layers of the `jetspotter` container image. Pull the `jetspotter` image.
```bash
docker pull ghcr.io/vvanouytsel/jetspotter:latest 
```

Fetch the manifest for linux and amd64.
```bash
crane manifest --platform linux/amd64 ghcr.io/vvanouytsel/jetspotter:latest | jq                 
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "digest": "sha256:b4d0f6cee04fa638a12f748043c33b9dae10bf4e1aab6d94c7619e568f09b092",
    "size": 1402
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:6a0ac1617861a677b045b7ff88545213ec31c0ff08763195a70a4a5adda577bb",
      "size": 3864189
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:9837455f32b31860f93d56fc50030167f6f7061f618eb18b4cbf5ac7345d0f9f",
      "size": 290252
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:4f4fb700ef54461cfa02571ae0db9a0dc1e0cdb5577484a6d75e68dc38e8acc1",
      "size": 32
    },
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "digest": "sha256:112001e747d0630a66960856820f8ba7a67756db73290b6a1b415edc9b1abca9",
      "size": 13551308
    }
  ]
}
```

As you can see now, we have 4 layers!
We can inspect these layers to see what they contain.

```bash
crane blob ghcr.io/vvanouytsel/jetspotter:latest@sha256:112001e747d0630a66960856820f8ba7a67756db73290b6a1b415edc9b1abca9 | tar -tzvf -

drwx------ 0/0               0 2026-05-21 15:08 root/
-rwxr-xr-x 0/0        24876149 2026-05-21 15:08 root/jetspotter
```

The above layer is the fourth layer. We can see that this layer includes the binary that we build.

Let's take a look at the [Dockerfile](https://github.com/vvanouytsel/jetspotter/blob/main/Dockerfile) of this project.

```Dockerfile
FROM golang:1.23 AS builder
WORKDIR /usr/src/app
COPY . /usr/src/app/

# Set build arguments for version information
ARG VERSION=dev
ARG COMMIT=unknown
ARG BUILD_TIME=unknown

# Build with CGO disabled for a fully static binary
ENV CGO_ENABLED=0

# Build with version information
RUN go build -o jetspotter -ldflags "-X jetspotter/internal/version.Version=${VERSION} -X jetspotter/internal/version.Commit=${COMMIT} -X jetspotter/internal/version.BuildTime=${BUILD_TIME}" cmd/jetspotter/jetspotter.go
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /usr/src/app/jetspotter .
CMD ["./jetspotter"]
```

This is a multi-stage build, where a build stage builds an artifact and then the artifact is copied over to another stage.

For the purpose of layers, we can simplify the Dockerfile to just have a look at the second stage, as that is the container that will be published.

```Dockerfile
FROM alpine:latest                               # Layer 1
RUN apk --no-cache add ca-certificates           # Layer 2
WORKDIR /root/                                   # Layer 3
COPY --from=builder /usr/src/app/jetspotter .    # Layer 4
CMD ["./jetspotter"]
```

Everything except the `CMD` (this only modifies metadata) creates a layer. Meaning this will create 4 layers in total. Yep, exactly those 4 layers we saw earlier. As you can see, the fourth layer does indeed contain the build golang artifact.

Let's check another layer. Let's check the 2nd layer. According to the manifest we fetched earlier, the digest of that layer is `sha256:6a0ac1617861a677b045b7ff88545213ec31c0ff08763195a70a4a5adda577bb`. So we can fetch it via crane, unpack it and list the contents.

If our analysis is correct, the second layer should contain the added file after installing the `ca-certificates` package.
```Dockerfile
RUN apk --no-cache add ca-certificates           # Layer 2
```

Yes, indeed. We can see a lot of files that were changed after installing the `ca-certificates` package.

```bash
crane blob ghcr.io/vvanouytsel/jetspotter:latest@sha256:9837455f32b31860f93d56fc50030167f6f7061f618eb18b4cbf5ac7345d0f9f | tar -tzvf -

drwxr-xr-x 0/0               0 2026-05-21 15:07 etc/
drwxr-xr-x 0/0               0 2026-05-21 15:07 etc/apk/
drwxr-xr-x 0/0               0 2026-05-21 15:07 etc/apk/protected_paths.d/
-rw-r--r-- 0/0             152 2026-04-14 21:21 etc/apk/protected_paths.d/ca-certificates.list
-rw-r--r-- 0/0              90 2026-05-21 15:07 etc/apk/world
drwxr-xr-x 0/0               0 2026-05-21 15:07 etc/ca-certificates/
drwxr-xr-x 0/0               0 2026-05-21 15:07 etc/ca-certificates/update.d/
-rwxr-xr-x 0/0              48 2026-04-14 21:21 etc/ca-certificates/update.d/certhash
-rw-r--r-- 0/0            5823 2026-04-14 21:21 etc/ca-certificates.conf
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/ssl/
drwxr-xr-x 0/0               0 2026-05-21 15:07 etc/ssl/certs/
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/002c0b4f.0 -> ca-cert-GlobalSign_Root_R46.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/0179095f.0 -> ca-cert-BJCA_Global_Root_CA1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/02265526.0 -> ca-cert-Entrust_Root_Certification_Authority_-_G2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/062cdee6.0 -> ca-cert-GlobalSign_Root_CA_-_R3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/064e0aa9.0 -> ca-cert-QuoVadis_Root_CA_2_G3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/06dc52d5.0 -> ca-cert-SSL.com_EV_Root_Certification_Authority_RSA_R2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/09789157.0 -> ca-cert-Starfield_Services_Root_Certificate_Authority_-_G2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/0a775a30.0 -> ca-cert-GTS_Root_R3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/0b1b94ef.0 -> ca-cert-CFCA_EV_ROOT.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/0b9bc432.0 -> ca-cert-ISRG_Root_X2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/0bf05006.0 -> ca-cert-SSL.com_Root_Certification_Authority_ECC.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/0f5dc4f3.0 -> ca-cert-UCA_Extended_Validation_Root.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/0f6fa695.0 -> ca-cert-GDCA_TrustAUTH_R5_ROOT.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/1001acf7.0 -> ca-cert-GTS_Root_R1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/106f3e4d.0 -> ca-cert-Entrust_Root_Certification_Authority_-_EC1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/14bc7599.0 -> ca-cert-emSign_ECC_Root_CA_-_G3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/1cef98f5.0 -> ca-cert-TrustAsia_Global_Root_CA_G4.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/1d3472b9.0 -> ca-cert-GlobalSign_ECC_Root_CA_-_R5.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/1e08bfd1.0 -> ca-cert-IdenTrust_Public_Sector_Root_CA_1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/1e09d511.0 -> ca-cert-T-TeleSec_GlobalRoot_Class_2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/244b5494.0 -> ca-cert-DigiCert_High_Assurance_EV_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/2923b3f9.0 -> ca-cert-emSign_Root_CA_-_G1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/2ae6433e.0 -> ca-cert-CA_Disig_Root_R2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/2b349938.0 -> ca-cert-AffirmTrust_Commercial.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/2ccbdda3.0 -> ca-cert-TrustAsia_TLS_ECC_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/30e1580d.0 -> ca-cert-OISTE_Server_Root_RSA_G1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/32888f65.0 -> ca-cert-Hellenic_Academic_and_Research_Institutions_RootCA_2015.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/3513523f.0 -> ca-cert-DigiCert_Global_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/3bde41ac.0 -> ca-cert-Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/3e359ba6.0 -> ca-cert-BJCA_Global_Root_CA2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/3fb36b73.0 -> ca-cert-NAVER_Global_Root_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/40193066.0 -> ca-cert-Certum_Trusted_Network_CA_2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/4042bcee.0 -> ca-cert-ISRG_Root_X1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/40547a79.0 -> ca-cert-COMODO_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/406c9bb1.0 -> ca-cert-emSign_Root_CA_-_C1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/48bec511.0 -> ca-cert-Certum_Trusted_Network_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/4b718d9b.0 -> ca-cert-emSign_ECC_Root_CA_-_C3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/4bfab552.0 -> ca-cert-Starfield_Root_Certificate_Authority_-_G2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/4f316efb.0 -> ca-cert-SwissSign_Gold_CA_-_G2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/5443e9e3.0 -> ca-cert-T-TeleSec_GlobalRoot_Class_3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/54657681.0 -> ca-cert-Buypass_Class_2_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/5860aaa6.0 -> ca-cert-Security_Communication_ECC_RootCA1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/5931b5bc.0 -> ca-cert-D-TRUST_EV_Root_CA_1_2020.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/5cd81ad7.0 -> ca-cert-TeliaSonera_Root_CA_v1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/5f15c80c.0 -> ca-cert-TWCA_Global_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/5f618aec.0 -> ca-cert-certSIGN_Root_CA_G2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/607986c7.0 -> ca-cert-DigiCert_Global_Root_G2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/616816f6.0 -> ca-cert-SecureSign_Root_CA12.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/626dceaf.0 -> ca-cert-GTS_Root_R2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/6805c744.0 -> ca-cert-OISTE_Server_Root_ECC_G1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/68dd7389.0 -> ca-cert-Hongkong_Post_Root_CA_3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/6a9bdba3.0 -> ca-cert-SecureSign_Root_CA15.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/6b99d060.0 -> ca-cert-Entrust_Root_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/6d41d539.0 -> ca-cert-Amazon_Root_CA_2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/6fa5da56.0 -> ca-cert-SSL.com_Root_Certification_Authority_RSA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/749e9e03.0 -> ca-cert-QuoVadis_Root_CA_1_G3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/75d1b2ed.0 -> ca-cert-DigiCert_Trusted_Root_G4.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/76faf6c0.0 -> ca-cert-QuoVadis_Root_CA_3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/7719f463.0 -> ca-cert-Hellenic_Academic_and_Research_Institutions_ECC_RootCA_2015.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/773e07ad.0 -> ca-cert-OISTE_WISeKey_Global_Root_GC_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/7a3adc42.0 -> ca-cert-vTrus_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/7a780d93.0 -> ca-cert-Certainly_Root_R1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/7f3d5d1d.0 -> ca-cert-DigiCert_Assured_ID_Root_G3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/7fa05551.0 -> ca-cert-Telekom_Security_TLS_RSA_Root_2023.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/8160b96c.0 -> ca-cert-Microsec_e-Szigno_Root_CA_2009.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/8508e720.0 -> ca-cert-Certainly_Root_E1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/865fbdf9.0 -> ca-cert-SSL.com_TLS_ECC_Root_CA_2022.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/878d9bca.0 -> ca-cert-SecureSign_Root_CA14.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/8cb5ee0f.0 -> ca-cert-Amazon_Root_CA_3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/8d86cdd1.0 -> ca-cert-certSIGN_ROOT_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/8d89cda1.0 -> ca-cert-Microsoft_ECC_Root_Certificate_Authority_2017.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/8f103249.0 -> ca-cert-Telia_Root_CA_v2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9046744a.0 -> ca-cert-Sectigo_Public_Server_Authentication_Root_R46.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/90c5a3c8.0 -> ca-cert-HiPKI_Root_CA_-_G1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/930ac5d2.0 -> ca-cert-Actalis_Authentication_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/93bc0acc.0 -> ca-cert-AffirmTrust_Networking.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9482e63a.0 -> ca-cert-Certum_EC-384_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9846683b.0 -> ca-cert-DigiCert_TLS_ECC_P384_Root_G5.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/988a38cb.0 -> ca-cert-NetLock_Arany_=Class_Gold=_Főtanúsítvány.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9b46e03d.0 -> ca-cert-Atos_TrustedRoot_Root_CA_RSA_TLS_2021.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9b5697b0.0 -> ca-cert-Trustwave_Global_ECC_P256_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9bf03295.0 -> ca-cert-TrustAsia_Global_Root_CA_G3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9c8dfbd4.0 -> ca-cert-AffirmTrust_Premium_ECC.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9d04f354.0 -> ca-cert-DigiCert_Assured_ID_Root_G2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9e654b62.0 -> ca-cert-SwissSign_RSA_TLS_Root_CA_2022_-_1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9ef4a08a.0 -> ca-cert-D-TRUST_BR_Root_CA_1_2020.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/9f727ac7.0 -> ca-cert-HARICA_TLS_RSA_Root_CA_2021.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/a09a51ae.0 -> ca-cert-D-TRUST_EV_Root_CA_2_2023.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/a3418fda.0 -> ca-cert-GTS_Root_R4.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/a89d74c2.0 -> ca-cert-SSL.com_TLS_RSA_Root_CA_2022.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/a94d09e5.0 -> ca-cert-ACCVRAIZ1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/b0d5255e.0 -> ca-cert-TrustAsia_TLS_RSA_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/b0e59380.0 -> ca-cert-GlobalSign_ECC_Root_CA_-_R4.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/b1159c4c.0 -> ca-cert-DigiCert_Assured_ID_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/b433981b.0 -> ca-cert-ANF_Secure_Server_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/b66938e9.0 -> ca-cert-Secure_Global_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/b727005e.0 -> ca-cert-AffirmTrust_Premium.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/b7a5b843.0 -> ca-cert-TWCA_Root_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/b81b93f0.0 -> ca-cert-AC_RAIZ_FNMT-RCM_SERVIDORES_SEGUROS.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/b8d25de6.0 -> ca-cert-TWCA_CYBER_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ba8887ce.0 -> ca-cert-FIRMAPROFESIONAL_CA_ROOT-A_WEB.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/bf53fb88.0 -> ca-cert-Microsoft_RSA_Root_Certificate_Authority_2017.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/c01eb047.0 -> ca-cert-UCA_Global_G2_Root.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/c28a8a30.0 -> ca-cert-D-TRUST_Root_Class_3_CA_2_2009.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-ACCVRAIZ1.pem -> /usr/share/ca-certificates/mozilla/ACCVRAIZ1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-AC_RAIZ_FNMT-RCM.pem -> /usr/share/ca-certificates/mozilla/AC_RAIZ_FNMT-RCM.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-AC_RAIZ_FNMT-RCM_SERVIDORES_SEGUROS.pem -> /usr/share/ca-certificates/mozilla/AC_RAIZ_FNMT-RCM_SERVIDORES_SEGUROS.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-ANF_Secure_Server_Root_CA.pem -> /usr/share/ca-certificates/mozilla/ANF_Secure_Server_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Actalis_Authentication_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Actalis_Authentication_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-AffirmTrust_Commercial.pem -> /usr/share/ca-certificates/mozilla/AffirmTrust_Commercial.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-AffirmTrust_Networking.pem -> /usr/share/ca-certificates/mozilla/AffirmTrust_Networking.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-AffirmTrust_Premium.pem -> /usr/share/ca-certificates/mozilla/AffirmTrust_Premium.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-AffirmTrust_Premium_ECC.pem -> /usr/share/ca-certificates/mozilla/AffirmTrust_Premium_ECC.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Amazon_Root_CA_1.pem -> /usr/share/ca-certificates/mozilla/Amazon_Root_CA_1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Amazon_Root_CA_2.pem -> /usr/share/ca-certificates/mozilla/Amazon_Root_CA_2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Amazon_Root_CA_3.pem -> /usr/share/ca-certificates/mozilla/Amazon_Root_CA_3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Amazon_Root_CA_4.pem -> /usr/share/ca-certificates/mozilla/Amazon_Root_CA_4.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Atos_TrustedRoot_2011.pem -> /usr/share/ca-certificates/mozilla/Atos_TrustedRoot_2011.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Atos_TrustedRoot_Root_CA_ECC_TLS_2021.pem -> /usr/share/ca-certificates/mozilla/Atos_TrustedRoot_Root_CA_ECC_TLS_2021.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Atos_TrustedRoot_Root_CA_RSA_TLS_2021.pem -> /usr/share/ca-certificates/mozilla/Atos_TrustedRoot_Root_CA_RSA_TLS_2021.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.pem -> /usr/share/ca-certificates/mozilla/Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-BJCA_Global_Root_CA1.pem -> /usr/share/ca-certificates/mozilla/BJCA_Global_Root_CA1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-BJCA_Global_Root_CA2.pem -> /usr/share/ca-certificates/mozilla/BJCA_Global_Root_CA2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Buypass_Class_2_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Buypass_Class_2_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Buypass_Class_3_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Buypass_Class_3_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-CA_Disig_Root_R2.pem -> /usr/share/ca-certificates/mozilla/CA_Disig_Root_R2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-CFCA_EV_ROOT.pem -> /usr/share/ca-certificates/mozilla/CFCA_EV_ROOT.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-COMODO_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/COMODO_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-COMODO_ECC_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/COMODO_ECC_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-COMODO_RSA_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/COMODO_RSA_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Certainly_Root_E1.pem -> /usr/share/ca-certificates/mozilla/Certainly_Root_E1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Certainly_Root_R1.pem -> /usr/share/ca-certificates/mozilla/Certainly_Root_R1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Certigna.pem -> /usr/share/ca-certificates/mozilla/Certigna.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Certigna_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Certigna_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Certum_EC-384_CA.pem -> /usr/share/ca-certificates/mozilla/Certum_EC-384_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Certum_Trusted_Network_CA.pem -> /usr/share/ca-certificates/mozilla/Certum_Trusted_Network_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Certum_Trusted_Network_CA_2.pem -> /usr/share/ca-certificates/mozilla/Certum_Trusted_Network_CA_2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Certum_Trusted_Root_CA.pem -> /usr/share/ca-certificates/mozilla/Certum_Trusted_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-D-TRUST_BR_Root_CA_1_2020.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_BR_Root_CA_1_2020.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-D-TRUST_BR_Root_CA_2_2023.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_BR_Root_CA_2_2023.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-D-TRUST_EV_Root_CA_1_2020.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_EV_Root_CA_1_2020.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-D-TRUST_EV_Root_CA_2_2023.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_EV_Root_CA_2_2023.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-D-TRUST_Root_Class_3_CA_2_2009.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_Root_Class_3_CA_2_2009.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-D-TRUST_Root_Class_3_CA_2_EV_2009.pem -> /usr/share/ca-certificates/mozilla/D-TRUST_Root_Class_3_CA_2_EV_2009.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_Assured_ID_Root_CA.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Assured_ID_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_Assured_ID_Root_G2.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Assured_ID_Root_G2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_Assured_ID_Root_G3.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Assured_ID_Root_G3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_Global_Root_CA.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Global_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_Global_Root_G2.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Global_Root_G2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_Global_Root_G3.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Global_Root_G3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_High_Assurance_EV_Root_CA.pem -> /usr/share/ca-certificates/mozilla/DigiCert_High_Assurance_EV_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_TLS_ECC_P384_Root_G5.pem -> /usr/share/ca-certificates/mozilla/DigiCert_TLS_ECC_P384_Root_G5.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_TLS_RSA4096_Root_G5.pem -> /usr/share/ca-certificates/mozilla/DigiCert_TLS_RSA4096_Root_G5.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-DigiCert_Trusted_Root_G4.pem -> /usr/share/ca-certificates/mozilla/DigiCert_Trusted_Root_G4.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Entrust_Root_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Entrust_Root_Certification_Authority_-_EC1.pem -> /usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority_-_EC1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Entrust_Root_Certification_Authority_-_G2.pem -> /usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority_-_G2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-FIRMAPROFESIONAL_CA_ROOT-A_WEB.pem -> /usr/share/ca-certificates/mozilla/FIRMAPROFESIONAL_CA_ROOT-A_WEB.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GDCA_TrustAUTH_R5_ROOT.pem -> /usr/share/ca-certificates/mozilla/GDCA_TrustAUTH_R5_ROOT.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GLOBALTRUST_2020.pem -> /usr/share/ca-certificates/mozilla/GLOBALTRUST_2020.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GTS_Root_R1.pem -> /usr/share/ca-certificates/mozilla/GTS_Root_R1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GTS_Root_R2.pem -> /usr/share/ca-certificates/mozilla/GTS_Root_R2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GTS_Root_R3.pem -> /usr/share/ca-certificates/mozilla/GTS_Root_R3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GTS_Root_R4.pem -> /usr/share/ca-certificates/mozilla/GTS_Root_R4.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GlobalSign_ECC_Root_CA_-_R4.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_ECC_Root_CA_-_R4.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GlobalSign_ECC_Root_CA_-_R5.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_ECC_Root_CA_-_R5.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GlobalSign_Root_CA_-_R3.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_Root_CA_-_R3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GlobalSign_Root_CA_-_R6.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_Root_CA_-_R6.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GlobalSign_Root_E46.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_Root_E46.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-GlobalSign_Root_R46.pem -> /usr/share/ca-certificates/mozilla/GlobalSign_Root_R46.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Go_Daddy_Root_Certificate_Authority_-_G2.pem -> /usr/share/ca-certificates/mozilla/Go_Daddy_Root_Certificate_Authority_-_G2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-HARICA_TLS_ECC_Root_CA_2021.pem -> /usr/share/ca-certificates/mozilla/HARICA_TLS_ECC_Root_CA_2021.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-HARICA_TLS_RSA_Root_CA_2021.pem -> /usr/share/ca-certificates/mozilla/HARICA_TLS_RSA_Root_CA_2021.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Hellenic_Academic_and_Research_Institutions_ECC_RootCA_2015.pem -> /usr/share/ca-certificates/mozilla/Hellenic_Academic_and_Research_Institutions_ECC_RootCA_2015.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Hellenic_Academic_and_Research_Institutions_RootCA_2015.pem -> /usr/share/ca-certificates/mozilla/Hellenic_Academic_and_Research_Institutions_RootCA_2015.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-HiPKI_Root_CA_-_G1.pem -> /usr/share/ca-certificates/mozilla/HiPKI_Root_CA_-_G1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Hongkong_Post_Root_CA_3.pem -> /usr/share/ca-certificates/mozilla/Hongkong_Post_Root_CA_3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-ISRG_Root_X1.pem -> /usr/share/ca-certificates/mozilla/ISRG_Root_X1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-ISRG_Root_X2.pem -> /usr/share/ca-certificates/mozilla/ISRG_Root_X2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-IdenTrust_Commercial_Root_CA_1.pem -> /usr/share/ca-certificates/mozilla/IdenTrust_Commercial_Root_CA_1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-IdenTrust_Public_Sector_Root_CA_1.pem -> /usr/share/ca-certificates/mozilla/IdenTrust_Public_Sector_Root_CA_1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Izenpe.com.pem -> /usr/share/ca-certificates/mozilla/Izenpe.com.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Microsec_e-Szigno_Root_CA_2009.pem -> /usr/share/ca-certificates/mozilla/Microsec_e-Szigno_Root_CA_2009.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Microsoft_ECC_Root_Certificate_Authority_2017.pem -> /usr/share/ca-certificates/mozilla/Microsoft_ECC_Root_Certificate_Authority_2017.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Microsoft_RSA_Root_Certificate_Authority_2017.pem -> /usr/share/ca-certificates/mozilla/Microsoft_RSA_Root_Certificate_Authority_2017.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-NAVER_Global_Root_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/NAVER_Global_Root_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-NetLock_Arany_=Class_Gold=_Főtanúsítvány.pem -> /usr/share/ca-certificates/mozilla/NetLock_Arany_=Class_Gold=_Főtanúsítvány.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-OISTE_Server_Root_ECC_G1.pem -> /usr/share/ca-certificates/mozilla/OISTE_Server_Root_ECC_G1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-OISTE_Server_Root_RSA_G1.pem -> /usr/share/ca-certificates/mozilla/OISTE_Server_Root_RSA_G1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-OISTE_WISeKey_Global_Root_GB_CA.pem -> /usr/share/ca-certificates/mozilla/OISTE_WISeKey_Global_Root_GB_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-OISTE_WISeKey_Global_Root_GC_CA.pem -> /usr/share/ca-certificates/mozilla/OISTE_WISeKey_Global_Root_GC_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-QuoVadis_Root_CA_1_G3.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_1_G3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-QuoVadis_Root_CA_2.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-QuoVadis_Root_CA_2_G3.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_2_G3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-QuoVadis_Root_CA_3.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-QuoVadis_Root_CA_3_G3.pem -> /usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_3_G3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SSL.com_EV_Root_Certification_Authority_ECC.pem -> /usr/share/ca-certificates/mozilla/SSL.com_EV_Root_Certification_Authority_ECC.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SSL.com_EV_Root_Certification_Authority_RSA_R2.pem -> /usr/share/ca-certificates/mozilla/SSL.com_EV_Root_Certification_Authority_RSA_R2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SSL.com_Root_Certification_Authority_ECC.pem -> /usr/share/ca-certificates/mozilla/SSL.com_Root_Certification_Authority_ECC.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SSL.com_Root_Certification_Authority_RSA.pem -> /usr/share/ca-certificates/mozilla/SSL.com_Root_Certification_Authority_RSA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SSL.com_TLS_ECC_Root_CA_2022.pem -> /usr/share/ca-certificates/mozilla/SSL.com_TLS_ECC_Root_CA_2022.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SSL.com_TLS_RSA_Root_CA_2022.pem -> /usr/share/ca-certificates/mozilla/SSL.com_TLS_RSA_Root_CA_2022.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SZAFIR_ROOT_CA2.pem -> /usr/share/ca-certificates/mozilla/SZAFIR_ROOT_CA2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Sectigo_Public_Server_Authentication_Root_E46.pem -> /usr/share/ca-certificates/mozilla/Sectigo_Public_Server_Authentication_Root_E46.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Sectigo_Public_Server_Authentication_Root_R46.pem -> /usr/share/ca-certificates/mozilla/Sectigo_Public_Server_Authentication_Root_R46.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SecureSign_Root_CA12.pem -> /usr/share/ca-certificates/mozilla/SecureSign_Root_CA12.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SecureSign_Root_CA14.pem -> /usr/share/ca-certificates/mozilla/SecureSign_Root_CA14.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SecureSign_Root_CA15.pem -> /usr/share/ca-certificates/mozilla/SecureSign_Root_CA15.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SecureTrust_CA.pem -> /usr/share/ca-certificates/mozilla/SecureTrust_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Secure_Global_CA.pem -> /usr/share/ca-certificates/mozilla/Secure_Global_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Security_Communication_ECC_RootCA1.pem -> /usr/share/ca-certificates/mozilla/Security_Communication_ECC_RootCA1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Security_Communication_RootCA2.pem -> /usr/share/ca-certificates/mozilla/Security_Communication_RootCA2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Starfield_Root_Certificate_Authority_-_G2.pem -> /usr/share/ca-certificates/mozilla/Starfield_Root_Certificate_Authority_-_G2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Starfield_Services_Root_Certificate_Authority_-_G2.pem -> /usr/share/ca-certificates/mozilla/Starfield_Services_Root_Certificate_Authority_-_G2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SwissSign_Gold_CA_-_G2.pem -> /usr/share/ca-certificates/mozilla/SwissSign_Gold_CA_-_G2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-SwissSign_RSA_TLS_Root_CA_2022_-_1.pem -> /usr/share/ca-certificates/mozilla/SwissSign_RSA_TLS_Root_CA_2022_-_1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-T-TeleSec_GlobalRoot_Class_2.pem -> /usr/share/ca-certificates/mozilla/T-TeleSec_GlobalRoot_Class_2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-T-TeleSec_GlobalRoot_Class_3.pem -> /usr/share/ca-certificates/mozilla/T-TeleSec_GlobalRoot_Class_3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TUBITAK_Kamu_SM_SSL_Kok_Sertifikasi_-_Surum_1.pem -> /usr/share/ca-certificates/mozilla/TUBITAK_Kamu_SM_SSL_Kok_Sertifikasi_-_Surum_1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TWCA_CYBER_Root_CA.pem -> /usr/share/ca-certificates/mozilla/TWCA_CYBER_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TWCA_Global_Root_CA.pem -> /usr/share/ca-certificates/mozilla/TWCA_Global_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TWCA_Root_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/TWCA_Root_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Telekom_Security_TLS_ECC_Root_2020.pem -> /usr/share/ca-certificates/mozilla/Telekom_Security_TLS_ECC_Root_2020.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Telekom_Security_TLS_RSA_Root_2023.pem -> /usr/share/ca-certificates/mozilla/Telekom_Security_TLS_RSA_Root_2023.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TeliaSonera_Root_CA_v1.pem -> /usr/share/ca-certificates/mozilla/TeliaSonera_Root_CA_v1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Telia_Root_CA_v2.pem -> /usr/share/ca-certificates/mozilla/Telia_Root_CA_v2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TrustAsia_Global_Root_CA_G3.pem -> /usr/share/ca-certificates/mozilla/TrustAsia_Global_Root_CA_G3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TrustAsia_Global_Root_CA_G4.pem -> /usr/share/ca-certificates/mozilla/TrustAsia_Global_Root_CA_G4.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TrustAsia_TLS_ECC_Root_CA.pem -> /usr/share/ca-certificates/mozilla/TrustAsia_TLS_ECC_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TrustAsia_TLS_RSA_Root_CA.pem -> /usr/share/ca-certificates/mozilla/TrustAsia_TLS_RSA_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Trustwave_Global_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/Trustwave_Global_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Trustwave_Global_ECC_P256_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/Trustwave_Global_ECC_P256_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-Trustwave_Global_ECC_P384_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/Trustwave_Global_ECC_P384_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-TunTrust_Root_CA.pem -> /usr/share/ca-certificates/mozilla/TunTrust_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-UCA_Extended_Validation_Root.pem -> /usr/share/ca-certificates/mozilla/UCA_Extended_Validation_Root.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-UCA_Global_G2_Root.pem -> /usr/share/ca-certificates/mozilla/UCA_Global_G2_Root.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-USERTrust_ECC_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/USERTrust_ECC_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-USERTrust_RSA_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/USERTrust_RSA_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-certSIGN_ROOT_CA.pem -> /usr/share/ca-certificates/mozilla/certSIGN_ROOT_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-certSIGN_Root_CA_G2.pem -> /usr/share/ca-certificates/mozilla/certSIGN_Root_CA_G2.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-e-Szigno_Root_CA_2017.pem -> /usr/share/ca-certificates/mozilla/e-Szigno_Root_CA_2017.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-e-Szigno_TLS_Root_CA_2023.pem -> /usr/share/ca-certificates/mozilla/e-Szigno_TLS_Root_CA_2023.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-ePKI_Root_Certification_Authority.pem -> /usr/share/ca-certificates/mozilla/ePKI_Root_Certification_Authority.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-emSign_ECC_Root_CA_-_C3.pem -> /usr/share/ca-certificates/mozilla/emSign_ECC_Root_CA_-_C3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-emSign_ECC_Root_CA_-_G3.pem -> /usr/share/ca-certificates/mozilla/emSign_ECC_Root_CA_-_G3.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-emSign_Root_CA_-_C1.pem -> /usr/share/ca-certificates/mozilla/emSign_Root_CA_-_C1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-emSign_Root_CA_-_G1.pem -> /usr/share/ca-certificates/mozilla/emSign_Root_CA_-_G1.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-vTrus_ECC_Root_CA.pem -> /usr/share/ca-certificates/mozilla/vTrus_ECC_Root_CA.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca-cert-vTrus_Root_CA.pem -> /usr/share/ca-certificates/mozilla/vTrus_Root_CA.crt
-rw-r--r-- 0/0          217769 2026-05-21 15:07 etc/ssl/certs/ca-certificates.crt
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ca6e4ad9.0 -> ca-cert-ePKI_Root_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/cbf06781.0 -> ca-cert-Go_Daddy_Root_Certificate_Authority_-_G2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/cc450945.0 -> ca-cert-Izenpe.com.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/cd58d51e.0 -> ca-cert-Security_Communication_RootCA2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/cd8c0d63.0 -> ca-cert-AC_RAIZ_FNMT-RCM.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ce5e74ef.0 -> ca-cert-Amazon_Root_CA_1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/d4dae3dd.0 -> ca-cert-D-TRUST_Root_Class_3_CA_2_EV_2009.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/d52c538d.0 -> ca-cert-DigiCert_TLS_RSA4096_Root_G5.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/d6325660.0 -> ca-cert-COMODO_RSA_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/d7e8dc79.0 -> ca-cert-QuoVadis_Root_CA_2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/d887a5bb.0 -> ca-cert-Trustwave_Global_ECC_P384_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/da0cfd1d.0 -> ca-cert-Sectigo_Public_Server_Authentication_Root_E46.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/dc4d6a89.0 -> ca-cert-GlobalSign_Root_CA_-_R6.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/dd8e9d41.0 -> ca-cert-DigiCert_Global_Root_G3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ddcda989.0 -> ca-cert-Telekom_Security_TLS_ECC_Root_2020.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/de6d66f3.0 -> ca-cert-Amazon_Root_CA_4.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/e113c810.0 -> ca-cert-Certigna.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/e18bfb83.0 -> ca-cert-QuoVadis_Root_CA_3_G3.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/e35234b1.0 -> ca-cert-Certum_Trusted_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/e36a6752.0 -> ca-cert-Atos_TrustedRoot_2011.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/e73d606e.0 -> ca-cert-OISTE_WISeKey_Global_Root_GB_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/e868b802.0 -> ca-cert-e-Szigno_Root_CA_2017.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/e8de2f56.0 -> ca-cert-Buypass_Class_3_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ecccd8db.0 -> ca-cert-HARICA_TLS_ECC_Root_CA_2021.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ed858448.0 -> ca-cert-vTrus_ECC_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/eed8c118.0 -> ca-cert-COMODO_ECC_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ef954a4e.0 -> ca-cert-IdenTrust_Commercial_Root_CA_1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/f0c70a8d.0 -> ca-cert-SSL.com_EV_Root_Certification_Authority_ECC.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/f249de83.0 -> ca-cert-Trustwave_Global_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/f30dd6ad.0 -> ca-cert-USERTrust_ECC_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/f39fc864.0 -> ca-cert-SecureTrust_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/f44703f1.0 -> ca-cert-e-Szigno_TLS_Root_CA_2023.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/f51bb24c.0 -> ca-cert-Certigna_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/fa5da96b.0 -> ca-cert-GLOBALTRUST_2020.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/fb717492.0 -> ca-cert-Atos_TrustedRoot_Root_CA_ECC_TLS_2021.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/fc5a8f99.0 -> ca-cert-USERTrust_RSA_Certification_Authority.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/fd64f3fc.0 -> ca-cert-TunTrust_Root_CA.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/fe8a2cd8.0 -> ca-cert-SZAFIR_ROOT_CA2.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/feffd413.0 -> ca-cert-GlobalSign_Root_E46.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ff34af3f.0 -> ca-cert-TUBITAK_Kamu_SM_SSL_Kok_Sertifikasi_-_Surum_1.pem
lrwxrwxrwx 0/0               0 2026-05-21 15:07 etc/ssl/certs/ffdd40f9.0 -> ca-cert-D-TRUST_BR_Root_CA_2_2023.pem
drwxr-xr-x 0/0               0 2026-04-15 06:51 lib/
drwxr-xr-x 0/0               0 2026-04-15 06:51 lib/apk/
drwxr-xr-x 0/0               0 2026-05-21 15:07 lib/apk/db/
-rw-r--r-- 0/0           25871 2026-05-21 15:07 lib/apk/db/installed
-rw------- 0/0               0 2026-05-21 15:07 lib/apk/db/lock
-rw-r--r-- 0/0            2091 2026-05-21 15:07 lib/apk/db/scripts.tar.gz
-rw-r--r-- 0/0             231 2026-05-21 15:07 lib/apk/db/triggers
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/
drwxr-xr-x 0/0               0 2026-05-21 15:07 usr/bin/
-rwxr-xr-x 0/0           14392 2026-04-14 21:21 usr/bin/c_rehash
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/local/
drwxr-xr-x 0/0               0 2026-05-21 15:07 usr/local/share/
drwxr-xr-x 0/0               0 2026-05-21 15:07 usr/local/share/ca-certificates/
drwxr-xr-x 0/0               0 2026-05-21 15:07 usr/sbin/
-rwxr-xr-x 0/0           14224 2026-04-14 21:21 usr/sbin/update-ca-certificates
drwxr-xr-x 0/0               0 2026-05-21 15:07 usr/share/
drwxr-xr-x 0/0               0 2026-05-21 15:07 usr/share/ca-certificates/
drwxr-xr-x 0/0               0 2026-05-21 15:07 usr/share/ca-certificates/mozilla/
-rw-r--r-- 0/0            2772 2026-04-14 21:21 usr/share/ca-certificates/mozilla/ACCVRAIZ1.crt
-rw-r--r-- 0/0            1972 2026-04-14 21:21 usr/share/ca-certificates/mozilla/AC_RAIZ_FNMT-RCM.crt
-rw-r--r-- 0/0             904 2026-04-14 21:21 usr/share/ca-certificates/mozilla/AC_RAIZ_FNMT-RCM_SERVIDORES_SEGUROS.crt
-rw-r--r-- 0/0            2118 2026-04-14 21:21 usr/share/ca-certificates/mozilla/ANF_Secure_Server_Root_CA.crt
-rw-r--r-- 0/0            2049 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Actalis_Authentication_Root_CA.crt
-rw-r--r-- 0/0            1204 2026-04-14 21:21 usr/share/ca-certificates/mozilla/AffirmTrust_Commercial.crt
-rw-r--r-- 0/0            1204 2026-04-14 21:21 usr/share/ca-certificates/mozilla/AffirmTrust_Networking.crt
-rw-r--r-- 0/0            1891 2026-04-14 21:21 usr/share/ca-certificates/mozilla/AffirmTrust_Premium.crt
-rw-r--r-- 0/0             753 2026-04-14 21:21 usr/share/ca-certificates/mozilla/AffirmTrust_Premium_ECC.crt
-rw-r--r-- 0/0            1188 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Amazon_Root_CA_1.crt
-rw-r--r-- 0/0            1883 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Amazon_Root_CA_2.crt
-rw-r--r-- 0/0             656 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Amazon_Root_CA_3.crt
-rw-r--r-- 0/0             737 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Amazon_Root_CA_4.crt
-rw-r--r-- 0/0            1261 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Atos_TrustedRoot_2011.crt
-rw-r--r-- 0/0             782 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Atos_TrustedRoot_Root_CA_ECC_TLS_2021.crt
-rw-r--r-- 0/0            1931 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Atos_TrustedRoot_Root_CA_RSA_TLS_2021.crt
-rw-r--r-- 0/0            2167 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Autoridad_de_Certificacion_Firmaprofesional_CIF_A62634068.crt
-rw-r--r-- 0/0            1952 2026-04-14 21:21 usr/share/ca-certificates/mozilla/BJCA_Global_Root_CA1.crt
-rw-r--r-- 0/0             806 2026-04-14 21:21 usr/share/ca-certificates/mozilla/BJCA_Global_Root_CA2.crt
-rw-r--r-- 0/0            1915 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Buypass_Class_2_Root_CA.crt
-rw-r--r-- 0/0            1915 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Buypass_Class_3_Root_CA.crt
-rw-r--r-- 0/0            1935 2026-04-14 21:21 usr/share/ca-certificates/mozilla/CA_Disig_Root_R2.crt
-rw-r--r-- 0/0            1984 2026-04-14 21:21 usr/share/ca-certificates/mozilla/CFCA_EV_ROOT.crt
-rw-r--r-- 0/0            1489 2026-04-14 21:21 usr/share/ca-certificates/mozilla/COMODO_Certification_Authority.crt
-rw-r--r-- 0/0             940 2026-04-14 21:21 usr/share/ca-certificates/mozilla/COMODO_ECC_Certification_Authority.crt
-rw-r--r-- 0/0            2086 2026-04-14 21:21 usr/share/ca-certificates/mozilla/COMODO_RSA_Certification_Authority.crt
-rw-r--r-- 0/0             741 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Certainly_Root_E1.crt
-rw-r--r-- 0/0            1891 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Certainly_Root_R1.crt
-rw-r--r-- 0/0            1330 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Certigna.crt
-rw-r--r-- 0/0            2264 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Certigna_Root_CA.crt
-rw-r--r-- 0/0             891 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Certum_EC-384_CA.crt
-rw-r--r-- 0/0            1354 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Certum_Trusted_Network_CA.crt
-rw-r--r-- 0/0            2078 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Certum_Trusted_Network_CA_2.crt
-rw-r--r-- 0/0            2053 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Certum_Trusted_Root_CA.crt
-rw-r--r-- 0/0            1050 2026-04-14 21:21 usr/share/ca-certificates/mozilla/D-TRUST_BR_Root_CA_1_2020.crt
-rw-r--r-- 0/0            2025 2026-04-14 21:21 usr/share/ca-certificates/mozilla/D-TRUST_BR_Root_CA_2_2023.crt
-rw-r--r-- 0/0            1050 2026-04-14 21:21 usr/share/ca-certificates/mozilla/D-TRUST_EV_Root_CA_1_2020.crt
-rw-r--r-- 0/0            2025 2026-04-14 21:21 usr/share/ca-certificates/mozilla/D-TRUST_EV_Root_CA_2_2023.crt
-rw-r--r-- 0/0            1517 2026-04-14 21:21 usr/share/ca-certificates/mozilla/D-TRUST_Root_Class_3_CA_2_2009.crt
-rw-r--r-- 0/0            1537 2026-04-14 21:21 usr/share/ca-certificates/mozilla/D-TRUST_Root_Class_3_CA_2_EV_2009.crt
-rw-r--r-- 0/0            1350 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_Assured_ID_Root_CA.crt
-rw-r--r-- 0/0            1306 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_Assured_ID_Root_G2.crt
-rw-r--r-- 0/0             851 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_Assured_ID_Root_G3.crt
-rw-r--r-- 0/0            1338 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_Global_Root_CA.crt
-rw-r--r-- 0/0            1294 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_Global_Root_G2.crt
-rw-r--r-- 0/0             839 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_Global_Root_G3.crt
-rw-r--r-- 0/0            1367 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_High_Assurance_EV_Root_CA.crt
-rw-r--r-- 0/0             790 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_TLS_ECC_P384_Root_G5.crt
-rw-r--r-- 0/0            1931 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_TLS_RSA4096_Root_G5.crt
-rw-r--r-- 0/0            1988 2026-04-14 21:21 usr/share/ca-certificates/mozilla/DigiCert_Trusted_Root_G4.crt
-rw-r--r-- 0/0            1643 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority.crt
-rw-r--r-- 0/0            1090 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority_-_EC1.crt
-rw-r--r-- 0/0            1533 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Entrust_Root_Certification_Authority_-_G2.crt
-rw-r--r-- 0/0             920 2026-04-14 21:21 usr/share/ca-certificates/mozilla/FIRMAPROFESIONAL_CA_ROOT-A_WEB.crt
-rw-r--r-- 0/0            1980 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GDCA_TrustAUTH_R5_ROOT.crt
-rw-r--r-- 0/0            1972 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GLOBALTRUST_2020.crt
-rw-r--r-- 0/0            1911 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GTS_Root_R1.crt
-rw-r--r-- 0/0            1911 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GTS_Root_R2.crt
-rw-r--r-- 0/0             765 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GTS_Root_R3.crt
-rw-r--r-- 0/0             765 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GTS_Root_R4.crt
-rw-r--r-- 0/0             704 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GlobalSign_ECC_Root_CA_-_R4.crt
-rw-r--r-- 0/0             794 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GlobalSign_ECC_Root_CA_-_R5.crt
-rw-r--r-- 0/0            1229 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GlobalSign_Root_CA_-_R3.crt
-rw-r--r-- 0/0            1972 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GlobalSign_Root_CA_-_R6.crt
-rw-r--r-- 0/0             769 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GlobalSign_Root_E46.crt
-rw-r--r-- 0/0            1915 2026-04-14 21:21 usr/share/ca-certificates/mozilla/GlobalSign_Root_R46.crt
-rw-r--r-- 0/0            1367 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Go_Daddy_Root_Certificate_Authority_-_G2.crt
-rw-r--r-- 0/0             867 2026-04-14 21:21 usr/share/ca-certificates/mozilla/HARICA_TLS_ECC_Root_CA_2021.crt
-rw-r--r-- 0/0            2017 2026-04-14 21:21 usr/share/ca-certificates/mozilla/HARICA_TLS_RSA_Root_CA_2021.crt
-rw-r--r-- 0/0            1017 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Hellenic_Academic_and_Research_Institutions_ECC_RootCA_2015.crt
-rw-r--r-- 0/0            2155 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Hellenic_Academic_and_Research_Institutions_RootCA_2015.crt
-rw-r--r-- 0/0            1939 2026-04-14 21:21 usr/share/ca-certificates/mozilla/HiPKI_Root_CA_-_G1.crt
-rw-r--r-- 0/0            2074 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Hongkong_Post_Root_CA_3.crt
-rw-r--r-- 0/0            1939 2026-04-14 21:21 usr/share/ca-certificates/mozilla/ISRG_Root_X1.crt
-rw-r--r-- 0/0             790 2026-04-14 21:21 usr/share/ca-certificates/mozilla/ISRG_Root_X2.crt
-rw-r--r-- 0/0            1923 2026-04-14 21:21 usr/share/ca-certificates/mozilla/IdenTrust_Commercial_Root_CA_1.crt
-rw-r--r-- 0/0            1931 2026-04-14 21:21 usr/share/ca-certificates/mozilla/IdenTrust_Public_Sector_Root_CA_1.crt
-rw-r--r-- 0/0            2122 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Izenpe.com.crt
-rw-r--r-- 0/0            1460 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Microsec_e-Szigno_Root_CA_2009.crt
-rw-r--r-- 0/0             875 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Microsoft_ECC_Root_Certificate_Authority_2017.crt
-rw-r--r-- 0/0            2021 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Microsoft_RSA_Root_Certificate_Authority_2017.crt
-rw-r--r-- 0/0            2013 2026-04-14 21:21 usr/share/ca-certificates/mozilla/NAVER_Global_Root_Certification_Authority.crt
-rw-r--r-- 0/0            1476 2026-04-14 21:21 usr/share/ca-certificates/mozilla/NetLock_Arany_=Class_Gold=_Főtanúsítvány.crt
-rw-r--r-- 0/0             826 2026-04-14 21:21 usr/share/ca-certificates/mozilla/OISTE_Server_Root_ECC_G1.crt
-rw-r--r-- 0/0            1972 2026-04-14 21:21 usr/share/ca-certificates/mozilla/OISTE_Server_Root_RSA_G1.crt
-rw-r--r-- 0/0            1346 2026-04-14 21:21 usr/share/ca-certificates/mozilla/OISTE_WISeKey_Global_Root_GB_CA.crt
-rw-r--r-- 0/0             895 2026-04-14 21:21 usr/share/ca-certificates/mozilla/OISTE_WISeKey_Global_Root_GC_CA.crt
-rw-r--r-- 0/0            1923 2026-04-14 21:21 usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_1_G3.crt
-rw-r--r-- 0/0            2041 2026-04-14 21:21 usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_2.crt
-rw-r--r-- 0/0            1923 2026-04-14 21:21 usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_2_G3.crt
-rw-r--r-- 0/0            2354 2026-04-14 21:21 usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_3.crt
-rw-r--r-- 0/0            1923 2026-04-14 21:21 usr/share/ca-certificates/mozilla/QuoVadis_Root_CA_3_G3.crt
-rw-r--r-- 0/0             956 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SSL.com_EV_Root_Certification_Authority_ECC.crt
-rw-r--r-- 0/0            2114 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SSL.com_EV_Root_Certification_Authority_RSA_R2.crt
-rw-r--r-- 0/0             944 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SSL.com_Root_Certification_Authority_ECC.crt
-rw-r--r-- 0/0            2094 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SSL.com_Root_Certification_Authority_RSA.crt
-rw-r--r-- 0/0             834 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SSL.com_TLS_ECC_Root_CA_2022.crt
-rw-r--r-- 0/0            1980 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SSL.com_TLS_RSA_Root_CA_2022.crt
-rw-r--r-- 0/0            1257 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SZAFIR_ROOT_CA2.crt
-rw-r--r-- 0/0             834 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Sectigo_Public_Server_Authentication_Root_E46.crt
-rw-r--r-- 0/0            1980 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Sectigo_Public_Server_Authentication_Root_R46.crt
-rw-r--r-- 0/0            1257 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SecureSign_Root_CA12.crt
-rw-r--r-- 0/0            1948 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SecureSign_Root_CA14.crt
-rw-r--r-- 0/0             802 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SecureSign_Root_CA15.crt
-rw-r--r-- 0/0            1350 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SecureTrust_CA.crt
-rw-r--r-- 0/0            1354 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Secure_Global_CA.crt
-rw-r--r-- 0/0             830 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Security_Communication_ECC_RootCA1.crt
-rw-r--r-- 0/0            1261 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Security_Communication_RootCA2.crt
-rw-r--r-- 0/0            1399 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Starfield_Root_Certificate_Authority_-_G2.crt
-rw-r--r-- 0/0            1424 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Starfield_Services_Root_Certificate_Authority_-_G2.crt
-rw-r--r-- 0/0            2045 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SwissSign_Gold_CA_-_G2.crt
-rw-r--r-- 0/0            1992 2026-04-14 21:21 usr/share/ca-certificates/mozilla/SwissSign_RSA_TLS_Root_CA_2022_-_1.crt
-rw-r--r-- 0/0            1367 2026-04-14 21:21 usr/share/ca-certificates/mozilla/T-TeleSec_GlobalRoot_Class_2.crt
-rw-r--r-- 0/0            1367 2026-04-14 21:21 usr/share/ca-certificates/mozilla/T-TeleSec_GlobalRoot_Class_3.crt
-rw-r--r-- 0/0            1582 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TUBITAK_Kamu_SM_SSL_Kok_Sertifikasi_-_Surum_1.crt
-rw-r--r-- 0/0            1984 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TWCA_CYBER_Root_CA.crt
-rw-r--r-- 0/0            1883 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TWCA_Global_Root_CA.crt
-rw-r--r-- 0/0            1269 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TWCA_Root_Certification_Authority.crt
-rw-r--r-- 0/0             843 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Telekom_Security_TLS_ECC_Root_2020.crt
-rw-r--r-- 0/0            2037 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Telekom_Security_TLS_RSA_Root_2023.crt
-rw-r--r-- 0/0            1870 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TeliaSonera_Root_CA_v1.crt
-rw-r--r-- 0/0            1952 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Telia_Root_CA_v2.crt
-rw-r--r-- 0/0            2017 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TrustAsia_Global_Root_CA_G3.crt
-rw-r--r-- 0/0             871 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TrustAsia_Global_Root_CA_G4.crt
-rw-r--r-- 0/0             822 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TrustAsia_TLS_ECC_Root_CA.crt
-rw-r--r-- 0/0            1968 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TrustAsia_TLS_RSA_Root_CA.crt
-rw-r--r-- 0/0            2090 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Trustwave_Global_Certification_Authority.crt
-rw-r--r-- 0/0             883 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Trustwave_Global_ECC_P256_Certification_Authority.crt
-rw-r--r-- 0/0             969 2026-04-14 21:21 usr/share/ca-certificates/mozilla/Trustwave_Global_ECC_P384_Certification_Authority.crt
-rw-r--r-- 0/0            2037 2026-04-14 21:21 usr/share/ca-certificates/mozilla/TunTrust_Root_CA.crt
-rw-r--r-- 0/0            1915 2026-04-14 21:21 usr/share/ca-certificates/mozilla/UCA_Extended_Validation_Root.crt
-rw-r--r-- 0/0            1891 2026-04-14 21:21 usr/share/ca-certificates/mozilla/UCA_Global_G2_Root.crt
-rw-r--r-- 0/0             948 2026-04-14 21:21 usr/share/ca-certificates/mozilla/USERTrust_ECC_Certification_Authority.crt
-rw-r--r-- 0/0            2094 2026-04-14 21:21 usr/share/ca-certificates/mozilla/USERTrust_RSA_Certification_Authority.crt
-rw-r--r-- 0/0            1176 2026-04-14 21:21 usr/share/ca-certificates/mozilla/certSIGN_ROOT_CA.crt
-rw-r--r-- 0/0            1891 2026-04-14 21:21 usr/share/ca-certificates/mozilla/certSIGN_Root_CA_G2.crt
-rw-r--r-- 0/0             843 2026-04-14 21:21 usr/share/ca-certificates/mozilla/e-Szigno_Root_CA_2017.crt
-rw-r--r-- 0/0            1034 2026-04-14 21:21 usr/share/ca-certificates/mozilla/e-Szigno_TLS_Root_CA_2023.crt
-rw-r--r-- 0/0            2033 2026-04-14 21:21 usr/share/ca-certificates/mozilla/ePKI_Root_Certification_Authority.crt
-rw-r--r-- 0/0             814 2026-04-14 21:21 usr/share/ca-certificates/mozilla/emSign_ECC_Root_CA_-_C3.crt
-rw-r--r-- 0/0             859 2026-04-14 21:21 usr/share/ca-certificates/mozilla/emSign_ECC_Root_CA_-_G3.crt
-rw-r--r-- 0/0            1257 2026-04-14 21:21 usr/share/ca-certificates/mozilla/emSign_Root_CA_-_C1.crt
-rw-r--r-- 0/0            1302 2026-04-14 21:21 usr/share/ca-certificates/mozilla/emSign_Root_CA_-_G1.crt
-rw-r--r-- 0/0             774 2026-04-14 21:21 usr/share/ca-certificates/mozilla/vTrus_ECC_Root_CA.crt
-rw-r--r-- 0/0            1911 2026-04-14 21:21 usr/share/ca-certificates/mozilla/vTrus_Root_CA.crt
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/
drwxr-xr-x 0/0               0 2026-05-21 15:07 var/log/
-rw-r--r-- 0/0             268 2026-05-21 15:07 var/log/apk.log
```

Let's also have a look at the first layer of our container image. 
Based on the Dockerfile, this should contain the full filesystem of `alpine:latest`.

```Dockerfile
FROM alpine:latest                               # Layer 1
```

Indeed it does. It contains the entire alpine root filesystem.

```bash
crane blob ghcr.io/vvanouytsel/jetspotter:latest@sha256:6a0ac1617861a677b045b7ff88545213ec31c0ff08763195a70a4a5adda577bb | tar -tzvf -

drwxr-xr-x 0/0               0 2026-04-15 06:51 bin/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/arch -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/ash -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/base64 -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/bbconfig -> /bin/busybox
-rwxr-xr-x 0/0          804616 2025-12-16 15:19 bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/cat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/chattr -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/chgrp -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/chmod -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/chown -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/cp -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/date -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/dd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/df -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/dmesg -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/dnsdomainname -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/dumpkmap -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/echo -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/egrep -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/false -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/fatattr -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/fdflush -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/fgrep -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/fsync -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/getopt -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/grep -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/gunzip -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/gzip -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/hostname -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/ionice -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/iostat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/ipcalc -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/kbd_mode -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/kill -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/link -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/linux32 -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/linux64 -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/ln -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/login -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/ls -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/lsattr -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/lzop -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/makemime -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/mkdir -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/mknod -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/mktemp -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/more -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/mount -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/mountpoint -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/mpstat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/mv -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/netstat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/nice -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/pidof -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/ping -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/ping6 -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/pipe_progress -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/printenv -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/ps -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/pwd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/reformime -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/rev -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/rm -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/rmdir -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/run-parts -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/sed -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/setpriv -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/setserial -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/sh -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/sleep -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/stat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/stty -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/su -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/sync -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/tar -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/touch -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/true -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/umount -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/uname -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/usleep -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/watch -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 bin/zcat -> /bin/busybox
drwxr-xr-x 0/0               0 2026-04-15 06:51 dev/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/
-rw-r--r-- 0/0               7 2026-04-15 06:50 etc/alpine-release
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/apk/
-rw-r--r-- 0/0               7 2026-04-15 06:51 etc/apk/arch
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/apk/keys/
-rw-r--r-- 0/0             451 2025-09-18 19:13 etc/apk/keys/alpine-devel@lists.alpinelinux.org-4a6a0840.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 etc/apk/keys/alpine-devel@lists.alpinelinux.org-5261cecb.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 etc/apk/keys/alpine-devel@lists.alpinelinux.org-6165ee59.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/apk/protected_paths.d/
-rw-r--r-- 0/0             103 2026-04-15 06:51 etc/apk/repositories
-rw-r--r-- 0/0              74 2026-04-15 06:51 etc/apk/world
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/busybox-paths.d/
-rw-r--r-- 0/0            3986 2025-12-16 15:19 etc/busybox-paths.d/busybox
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/crontabs/
-rw------- 0/0             283 2026-03-25 08:02 etc/crontabs/root
-rw-r--r-- 0/0              89 2026-03-25 08:02 etc/fstab
-rw-r--r-- 0/0             510 2026-03-25 08:02 etc/group
-rw-r--r-- 0/0              10 2026-03-25 08:02 etc/hostname
-rw-r--r-- 0/0              79 2026-03-25 08:02 etc/hosts
-rw-r--r-- 0/0             570 2026-03-25 08:02 etc/inittab
-rw-r--r-- 0/0              51 2026-04-15 06:50 etc/issue
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/logrotate.d/
-rw-r--r-- 0/0             140 2025-12-16 15:19 etc/logrotate.d/acpid
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/modprobe.d/
-rw-r--r-- 0/0            1545 2026-03-25 08:02 etc/modprobe.d/aliases.conf
-rw-r--r-- 0/0            2247 2026-03-25 08:02 etc/modprobe.d/blacklist.conf
-rw-r--r-- 0/0             122 2026-03-25 08:02 etc/modprobe.d/i386.conf
-rw-r--r-- 0/0              15 2026-03-25 08:02 etc/modules
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/modules-load.d/
-rw-r--r-- 0/0             284 2026-03-25 08:02 etc/motd
lrwxrwxrwx 0/0               0 2026-04-15 06:51 etc/mtab -> ../proc/mounts
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/network/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/network/if-down.d/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/network/if-post-down.d/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/network/if-post-up.d/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/network/if-pre-down.d/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/network/if-pre-up.d/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/network/if-up.d/
-rwxrwxr-x 0/0             285 2025-12-16 15:19 etc/network/if-up.d/dad
-rw-r--r-- 0/0             205 2026-03-25 08:02 etc/nsswitch.conf
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/opt/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 etc/os-release -> ../usr/lib/os-release
-rw-r--r-- 0/0             702 2026-03-25 08:02 etc/passwd
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/periodic/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/periodic/15min/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/periodic/daily/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/periodic/hourly/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/periodic/monthly/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/periodic/weekly/
-rw-r--r-- 0/0             547 2026-03-25 08:02 etc/profile
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/profile.d/
-rw-r--r-- 0/0              97 2026-03-25 08:02 etc/profile.d/20locale.sh
-rw-r--r-- 0/0             249 2026-03-25 08:02 etc/profile.d/README
-rw-r--r-- 0/0             447 2026-03-25 08:02 etc/profile.d/color_prompt.sh.disabled
-rw-r--r-- 0/0            3144 2026-03-25 08:02 etc/protocols
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/secfixes.d/
-rw-r--r-- 0/0              97 2026-04-15 06:50 etc/secfixes.d/alpine
-rw-r--r-- 0/0             156 2025-12-16 15:19 etc/securetty
-rw-r--r-- 0/0           12813 2026-03-25 08:02 etc/services
-rw-r----- 0/42            260 2026-04-15 06:51 etc/shadow
-rw-r--r-- 0/0              38 2026-03-25 08:02 etc/shells
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/ssl/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 etc/ssl/cert.pem -> certs/ca-certificates.crt
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/ssl/certs/
-rw-r--r-- 0/0          217769 2026-04-14 21:21 etc/ssl/certs/ca-certificates.crt
-rw-r--r-- 0/0             412 2026-04-10 00:06 etc/ssl/ct_log_list.cnf
-rw-r--r-- 0/0             412 2026-04-10 00:06 etc/ssl/ct_log_list.cnf.dist
-rw-r--r-- 0/0           12411 2026-04-10 00:06 etc/ssl/openssl.cnf
-rw-r--r-- 0/0           12411 2026-04-10 00:06 etc/ssl/openssl.cnf.dist
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/ssl/private/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/ssl1.1/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 etc/ssl1.1/cert.pem -> /etc/ssl/cert.pem
lrwxrwxrwx 0/0               0 2026-04-15 06:51 etc/ssl1.1/certs -> /etc/ssl/certs
-rw-r--r-- 0/0              53 2026-03-25 08:02 etc/sysctl.conf
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/sysctl.d/
drwxr-xr-x 0/0               0 2026-04-15 06:51 etc/udhcpc/
-rw-r--r-- 0/0             287 2025-12-16 15:19 etc/udhcpc/udhcpc.conf
drwxr-xr-x 0/0               0 2026-04-15 06:51 home/
drwxr-xr-x 0/0               0 2026-04-15 06:51 lib/
drwxr-xr-x 0/0               0 2026-04-15 06:51 lib/apk/
drwxr-xr-x 0/0               0 2026-04-15 06:51 lib/apk/db/
-rw-r--r-- 0/0           15163 2026-04-15 06:51 lib/apk/db/installed
-rw-r--r-- 0/0            1916 2026-04-15 06:51 lib/apk/db/scripts.tar.gz
-rw-r--r-- 0/0              95 2026-04-15 06:51 lib/apk/db/triggers
drwxr-xr-x 0/0               0 2026-04-15 06:51 lib/apk/exec/
drwxr-xr-x 0/0               0 2026-04-15 06:51 lib/firmware/
-rwxr-xr-x 0/0          666216 2026-04-10 17:30 lib/ld-musl-x86_64.so.1
lrwxrwxrwx 0/0               0 2026-04-15 06:51 lib/libc.musl-x86_64.so.1 -> ld-musl-x86_64.so.1
drwxr-xr-x 0/0               0 2026-04-15 06:51 lib/modules-load.d/
drwxr-xr-x 0/0               0 2026-04-15 06:51 lib/sysctl.d/
drwxr-xr-x 0/0               0 2026-04-15 06:51 media/
drwxr-xr-x 0/0               0 2026-04-15 06:51 media/cdrom/
drwxr-xr-x 0/0               0 2026-04-15 06:51 media/floppy/
drwxr-xr-x 0/0               0 2026-04-15 06:51 media/usb/
drwxr-xr-x 0/0               0 2026-04-15 06:51 mnt/
drwxr-xr-x 0/0               0 2026-04-15 06:51 opt/
drwxr-xr-x 0/0               0 2026-04-15 06:51 proc/
drwx------ 0/0               0 2026-04-15 06:51 root/
drwxr-xr-x 0/0               0 2026-04-15 06:51 run/
drwxr-xr-x 0/0               0 2026-04-15 06:51 run/lock/
drwxr-xr-x 0/0               0 2026-04-15 06:51 sbin/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/acpid -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/adjtimex -> /bin/busybox
-rwxr-xr-x 0/0          115096 2026-04-14 16:06 sbin/apk
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/arp -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/blkdiscard -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/blkid -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/blockdev -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/depmod -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/fbsplash -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/fdisk -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/findfs -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/fsck -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/fstrim -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/getty -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/halt -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/hwclock -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/ifconfig -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/ifdown -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/ifenslave -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/ifup -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/init -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/inotifyd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/insmod -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/ip -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/ipaddr -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/iplink -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/ipneigh -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/iproute -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/iprule -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/iptunnel -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/klogd -> /bin/busybox
-rwxr-xr-x 0/0             393 2026-04-10 17:30 sbin/ldconfig
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/loadkmap -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/logread -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/losetup -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/lsmod -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/mdev -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/mkdosfs -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/mkfs.vfat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/mkswap -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/modinfo -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/modprobe -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/nameif -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/nologin -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/pivot_root -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/poweroff -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/raidautorun -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/reboot -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/rmmod -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/route -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/setconsole -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/slattach -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/swapoff -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/swapon -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/switch_root -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/sysctl -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/syslogd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/tunctl -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/udhcpc -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/vconfig -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/watchdog -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 sbin/zcip -> /bin/busybox
drwxr-xr-x 0/0               0 2026-04-15 06:51 srv/
drwxr-xr-x 0/0               0 2026-04-15 06:51 sys/
drwxrwxrwt 0/0               0 2026-04-15 06:51 tmp/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/bin/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/[ -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/[[ -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/awk -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/basename -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/bc -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/beep -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/bunzip2 -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/bzcat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/bzip2 -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/cal -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/chvt -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/cksum -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/clear -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/cmp -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/comm -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/cpio -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/crontab -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/cryptpw -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/cut -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/dc -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/deallocvt -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/diff -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/dirname -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/dos2unix -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/du -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/eject -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/env -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/expand -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/expr -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/factor -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/fallocate -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/find -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/flock -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/fold -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/free -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/fuser -> /bin/busybox
-rwxr-xr-x 0/0           22344 2026-04-10 17:30 usr/bin/getconf
-rwxr-xr-x 0/0           18480 2026-04-10 17:30 usr/bin/getent
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/groups -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/hd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/head -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/hexdump -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/hostid -> /bin/busybox
-rwxr-xr-x 0/0           14152 2026-04-10 17:30 usr/bin/iconv
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/id -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/install -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/ipcrm -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/ipcs -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/killall -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/last -> /bin/busybox
-rwxr-xr-x 0/0              52 2026-04-10 17:30 usr/bin/ldd
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/less -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/logger -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/lsof -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/lsusb -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/lzcat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/lzma -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/lzopcat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/md5sum -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/mesg -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/microcom -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/mkfifo -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/mkpasswd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/nc -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/nl -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/nmeter -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/nohup -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/nproc -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/nsenter -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/nslookup -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/od -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/openvt -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/passwd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/paste -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/pgrep -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/pkill -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/pmap -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/printf -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/pscan -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/pstree -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/pwdx -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/readlink -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/realpath -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/renice -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/reset -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/resize -> /bin/busybox
-rwxr-xr-x 0/0           67504 2025-07-15 09:26 usr/bin/scanelf
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/seq -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/setkeycodes -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/setsid -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/sha1sum -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/sha256sum -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/sha3sum -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/sha512sum -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/showkey -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/shred -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/shuf -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/sort -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/split -> /bin/busybox
-rwxr-xr-x 0/0           14384 2025-12-16 15:19 usr/bin/ssl_client
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/strings -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/sum -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/tac -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/tail -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/tee -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/test -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/time -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/timeout -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/top -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/tr -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/traceroute -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/traceroute6 -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/tree -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/truncate -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/tty -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/ttysize -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/udhcpc6 -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/unexpand -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/uniq -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/unix2dos -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/unlink -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/unlzma -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/unlzop -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/unshare -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/unxz -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/unzip -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/uptime -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/uudecode -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/uuencode -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/vi -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/vlock -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/volname -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/wc -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/wget -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/which -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/who -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/whoami -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/whois -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/xargs -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/xxd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/xzcat -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/bin/yes -> /bin/busybox
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/lib/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/lib/engines-3/
-rwxr-xr-x 0/0           18504 2026-04-10 00:06 usr/lib/engines-3/afalg.so
-rwxr-xr-x 0/0           13864 2026-04-10 00:06 usr/lib/engines-3/capi.so
-rwxr-xr-x 0/0           47608 2026-04-10 00:06 usr/lib/engines-3/loader_attic.so
-rwxr-xr-x 0/0           22360 2026-04-10 00:06 usr/lib/engines-3/padlock.so
-rwxr-xr-x 0/0          277184 2026-04-14 16:06 usr/lib/libapk.so.3.0.0
-rwxr-xr-x 0/0         4985616 2026-04-10 00:06 usr/lib/libcrypto.so.3
-rwxr-xr-x 0/0          839544 2026-04-10 00:06 usr/lib/libssl.so.3
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/lib/libz.so.1 -> libz.so.1.3.2
-rwxr-xr-x 0/0          108376 2026-03-06 08:33 usr/lib/libz.so.1.3.2
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/lib/modules-load.d/
-rw-r--r-- 0/0             188 2026-04-15 06:50 usr/lib/os-release
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/lib/ossl-modules/
-rwxr-xr-x 0/0          100184 2026-04-10 00:06 usr/lib/ossl-modules/legacy.so
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/lib/sysctl.d/
-rw-r--r-- 0/0            1278 2026-03-25 08:02 usr/lib/sysctl.d/00-alpine.conf
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/local/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/local/bin/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/local/lib/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/local/share/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/sbin/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/add-shell -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/addgroup -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/adduser -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/arping -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/brctl -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/chpasswd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/chroot -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/crond -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/delgroup -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/deluser -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/ether-wake -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/fbset -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/killall5 -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/loadfont -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/nanddump -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/nandwrite -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/nbd-client -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/ntpd -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/partprobe -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/rdate -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/rdev -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/readahead -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/remove-shell -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/rfkill -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/sendmail -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/setfont -> /bin/busybox
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/sbin/setlogcons -> /bin/busybox
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/aarch64/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/aarch64/alpine-devel@lists.alpinelinux.org-58199dcc.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-58199dcc.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/aarch64/alpine-devel@lists.alpinelinux.org-616ae350.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-616ae350.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-4a6a0840.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-5243ef4b.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-524d27bb.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-5261cecb.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-58199dcc.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-58cbb476.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-58e4f17d.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-5e69ca50.rsa.pub
-rw-r--r-- 0/0             451 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-60ac2099.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-6165ee59.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-61666e3f.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-616a9724.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-616abc23.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-616ac3bc.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-616adfeb.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-616ae350.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-616db30d.rsa.pub
-rw-r--r-- 0/0             800 2025-09-18 19:13 usr/share/apk/keys/alpine-devel@lists.alpinelinux.org-66ba20fe.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/armhf/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/armhf/alpine-devel@lists.alpinelinux.org-524d27bb.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-524d27bb.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/armhf/alpine-devel@lists.alpinelinux.org-616a9724.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-616a9724.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/armv7/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/armv7/alpine-devel@lists.alpinelinux.org-524d27bb.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-524d27bb.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/armv7/alpine-devel@lists.alpinelinux.org-616adfeb.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-616adfeb.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/loongarch64/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/loongarch64/alpine-devel@lists.alpinelinux.org-66ba20fe.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-66ba20fe.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/mips64/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/mips64/alpine-devel@lists.alpinelinux.org-5e69ca50.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-5e69ca50.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/ppc64le/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/ppc64le/alpine-devel@lists.alpinelinux.org-58cbb476.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-58cbb476.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/ppc64le/alpine-devel@lists.alpinelinux.org-616abc23.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-616abc23.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/riscv64/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/riscv64/alpine-devel@lists.alpinelinux.org-60ac2099.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-60ac2099.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/riscv64/alpine-devel@lists.alpinelinux.org-616db30d.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-616db30d.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/s390x/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/s390x/alpine-devel@lists.alpinelinux.org-58e4f17d.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-58e4f17d.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/s390x/alpine-devel@lists.alpinelinux.org-616ac3bc.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-616ac3bc.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/x86/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/x86/alpine-devel@lists.alpinelinux.org-4a6a0840.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-4a6a0840.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/x86/alpine-devel@lists.alpinelinux.org-5243ef4b.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-5243ef4b.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/x86/alpine-devel@lists.alpinelinux.org-61666e3f.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-61666e3f.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/apk/keys/x86_64/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/x86_64/alpine-devel@lists.alpinelinux.org-4a6a0840.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-4a6a0840.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/x86_64/alpine-devel@lists.alpinelinux.org-5261cecb.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-5261cecb.rsa.pub
lrwxrwxrwx 0/0               0 2026-04-15 06:51 usr/share/apk/keys/x86_64/alpine-devel@lists.alpinelinux.org-6165ee59.rsa.pub -> ../alpine-devel@lists.alpinelinux.org-6165ee59.rsa.pub
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/misc/
drwxr-xr-x 0/0               0 2026-04-15 06:51 usr/share/udhcpc/
-rwxr-xr-x 0/0            4010 2025-12-16 15:19 usr/share/udhcpc/default.script
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/cache/
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/cache/apk/
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/cache/misc/
dr-xr-xr-x 0/0               0 2026-04-15 06:51 var/empty/
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/lib/
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/lib/misc/
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/local/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 var/lock -> ../run/lock
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/log/
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/mail/
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/opt/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 var/run -> ../run
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/spool/
drwxr-xr-x 0/0               0 2026-04-15 06:51 var/spool/cron/
lrwxrwxrwx 0/0               0 2026-04-15 06:51 var/spool/cron/crontabs -> ../../../etc/crontabs
lrwxrwxrwx 0/0               0 2026-04-15 06:51 var/spool/mail -> ../mail
drwxrwxrwt 0/0               0 2026-04-15 06:51 var/tmp/
```

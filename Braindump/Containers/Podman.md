---
tags:
  - containers
---

## Authentication

You can use `login` to authenticate with a registry.

```bash
podman login my-repo.domain.com
```

By default your token is stored in `${XDG_RUNTIME_DIR}` and is removed on boot. If you want make it persistent you can use an `authfile`.

```bash
podman login --authfile $HOME/.config/containers/auth.json my-repo.domain.com
```

## User IDs

If a podman container is ran as root, it automatically maps the root user (0) to your host OS non-root user (1000). This is perfect as volumes simply 'work'.

If your container runs as a different user, for example '472', then this will be mapped to an offset.

```bash
 ❯ podman unshare cat /proc/self/uid_map
         0       1000          1
         1     524288      65536
```

In the above command you can see that the container UID of 0 is mapped to my local OS uid of 1000. Which is my regular user.

A UID of 1 in the container would be mapped to 524288.
A UID of 10 in the container would be mapped to 524298.

This can mess up your file permissions when you attempt to mount something from your host in your container. However, you can always specify that you want to run the container as the user 0.

```bash
podman run --user 0:0 -d --name grafana docker.io/grafana/grafana
```

**References**:
* [User IDs and (rootless) containers with Podman – Just another Linux geek](https://blog.christophersmart.com/2021/01/26/user-ids-and-rootless-containers-with-podman/)
* [Volumes and rootless Podman – Just another Linux geek](https://blog.christophersmart.com/2021/01/31/volumes-and-rootless-podman/)

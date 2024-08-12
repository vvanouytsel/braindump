#containers
## Authentication

You can use `login` to authenticate with a registry.

```bash
podman login my-repo.domain.com
```

By default your token is stored in `${XDG_RUNTIME_DIR}` and is removed on boot. If you want make it persistent you can use an `authfile`. 
```bash
podman login --authfile $HOME/.config/containers/auth.json my-repo.domain.com
```
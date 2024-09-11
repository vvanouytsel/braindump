

Use SSH tunneling to port forward `8080` on your local host to port `9090` of your target host

```bash
ssh -L localhost:8080:127.0.0.1:9090 my-my-example-server
```

For example

```bash
# Copy the kubeconfig file from remote to your host and run the following
ssh -fNT -L 6443:127.0.0.1:6443 eagle.vvanouytsel.com
```

To verify if a public key matches a private key

```bash
# If both hashes match, the public key is coupled to the private key
❯ ssh-keygen -lf ~/.ssh/id_ed25519
256 SHA256:XaRu8qDrytQtZLWCeUaugTAP17xAFEGATRi+FkCZCZc y509496@softwareag.com (ED25519)

❯ ssh-keygen -lf ~/.ssh/id_ed25519.pub
256 SHA256:XaRu8qDrytQtZLWCeUaugTAP17xAFEGATRi+FkCZCZc y509496@softwareag.com (ED25519)
```

## Jump host

You can specify a jump host when using SSH. Your SSH connection will pass through the specified jump host, but the private key on your local system will be used. This is a great alternative if you cannot reach a target server locally, but don't want to copy your private key to a jump host.

```bash
$ ssh user@my-targetserver -J my-jumpserver

# You can use the same principle with scp
$ scp -J my-jumpserver user@my-targetserver:/tmp/file /tmp
```
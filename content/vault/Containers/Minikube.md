#containers #tools 
- Run minikube with a specific kubernetes version

```bash
$ minikube start --profile minikube-1.24 --kubernetes-version=v1.24.0
```

- List all profiles

```bash
$ minikube profile list
```

- Open shell to cluster

```bash
$ minikube ssh --profile minikube-1.24
```

- Delete minicube cluster

```bash
$ minikube delete --profile minikube-1.24
```
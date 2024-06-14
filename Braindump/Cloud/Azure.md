#azure #cloud
## Authentication

### Username and Password

Log in to azure and use correct subscription

```bash
az account set --subscription 'Company'
az login
```

### Service Principals

Service principals are identities used by automated tooling to authenticate with Azure.  
Create a service principal

```bash
# Take note of the generated password, since it will not be retrievable anymore
az ad sp create-for-rbac --name ServicePrincipalName

# You can reset the generated password in case you lose it
az ad sp credential reset --name APP_ID
```

Delete a service principal

```bash
az ad sp delete  --id APP_ID
```

Log in using a service principal

```bash
az login --service-principal --username APP_ID --password PASSWORD --tenant TENANT_ID
```

## Virtual Machine

Create a virtual machine

```bash
az vm create \
      --resource-group rg-poc-pipeline-devops-azure \
      --location westeurope \
      --assign-identity \
      --scope /subscriptions/ec48b4cb-76f9-487b-8fea-35a53a7618be/resourceGroups/rg-poc-pipeline-devops-azure/providers/Microsoft.ContainerRegistry/registries/crpocpipelinedevopsazure \
      --name my-poc-azure-agent \
      --image images \
      --ssh-key-value ~/.ssh/id_rsa.pub \
      --public-ip-address "" \
      --private-ip-address "10.92.0.200" \
      --subnet /subscriptions/ec48b4cb-76f9-487b-8fea-35a53a7618be/resourceGroups/rg-development/providers/Microsoft.Network/virtualNetworks/vn-development/subnets/sn_development-aks

```

Delete a virtual machine

```bash
az vm delete --resource-group rg-poc-pipeline-devops-azure --name my-poc-azure-agent
```

### Identity

Create an identity for a VM

```bash
az vm identity assign --resource-group rg-poc-pipeline-devops-azure --name my-poc-azure-agent
```

Get the principalId of the identity

```bash
spID=$(az vm show --resource-group rg-poc-pipeline-devops-azure --name my-poc-azure-agent --query identity.principalId --out tsv)
```

Get the id of the resource you want to provide access to

```bash
resourceID=$(az acr show --resource-group rg-poc-pipeline-devops-azure --name crpocpipelinedevopsazure --query id --output tsv)
```

Provide pull and push access to the identity

```bash
az role assignment create --assignee $spID --scope $resourceID --role acrpush
```

Use the identity on the VM

```bash
az login --identity
```

Log in to the ACR

```bash
az acr login --name crpocpipelinedevopsazure
```

## Webapp

Show logs of a specific webapp

```bash
az webapp log tail --name my-admin-guide-staging --resource-group rg-pipeline
```

## Registry

### Container Registry

Show usage of a container registry

```bash
az acr show-usage --name crinfradocker

--------  ------------  ---------------  ------
Size      536870912000  2632557896       Bytes
Webhooks  500           1                Count
```

Show images ina a container registry

```bash
az acr repository list --name crpocpipelinedevopsazure
```

Show tags of an image in a container registry

```bash
az acr repository show-tags --name crinfradocker --repository my-admin-guide

Result
-----------
20191008.5
20191009.1
20191009.2
20191128.1
20200106.1
20200106.2
```

Delete tag of a image in a repository in a container registry

```bash
az acr repository delete --name crinfradocker --yes --image my-admin-guide:20191008.5
```

### Helm Registry

List helm charts in a registry

```bash
az acr helm list --name crpocpipelinedevopsazure
```

## DNS

Query dns zones

```bash
az network dns zone list
```

List dns A records

```bash
az network dns record-set a list --zone-name company.com --resource-group rg-dns-development
```

List dns CNAME records

```bash
az network dns record-set cname list --zone-name company.com --resource-group rg-dns-development
```

## Ask

Store credentials in local kubeconfig

```bash
az aks get-credentials --resource-group rg-poc-pipeline-devops-azure --name akspocpipelinedevopsazure
```

Set the default namespace

```bash
kubectl config set-context --current --namespace=<insert-namespace-name-here>
```

Verify the default namespace

```bash
kubectl config view --minify | grep namespace:
```

Access the Kubernetes dashboard

```bash
# Proxy all local requests to k8s cluster
kubectl proxy

# Navigate to the dashboard using the browser
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

Integrate Ask with ACR so it can pull images from the registry

```sh
az aks update -n akspocpipelinedevopsazure -g rg-poc-pipeline-devops-azure --attach-acr crpocpipelinedevopsazure
```

### Ingress Controller

Create a file `ingress-internal.yml` with the following content

```yaml
controller:
  service:
    loadBalancerIP: 10.92.0.196
    annotations:
      service.beta.kubernetes.io/azure-load-balancer-internal: 'true'
```

Deploy the ingress helm chart

```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
kubectl create namespace ingress
helm install ingress stable/nginx-ingress -f ingress-internal.yml --set controller.replicaCount=1 -n ingress
```

Verify the `nginx-ingress-controller` gets an external-ip

```bash
# Note: A loadbalancer named 'kubernetes-internal' will be automatically created in Azure
kubectl get service -w -n ingress

NAME                                    TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-ingress-controller        LoadBalancer   10.0.235.155   10.92.0.196   80:30354/TCP,443:30142/TCP   124m
ingress-nginx-ingress-default-backend   ClusterIP      10.0.179.91    <none>        80/TCP                       124m
```

### Ask and ACR Integration

Integrate existing ACR with existing Ask

```bash
ACR_NAME=your-acr-name
ACR_RESOURCE_GROUP=your-acr-ressource-group

ACR_ID=$(az acr show --name $ACR_NAME \
     --resource-group $ACR_RESOURCE_GROUP \
     --query "id" --output tsv)

az aks update --name your-aks-name \
    --resource-group your-aks-resource-group-name \
    --attach-acr $ACR_ID
```

### Helm

Show templated resources

```bash
helm install myApp  . --dry-run
```

Search, download and unpack a helm chart

```bash
# Search for the helm chart
helm search repo my-test-flow

NAME                                  CHART VERSION APP VERSION DESCRIPTION
crpocpipelinedevopsazure/my-test-flow 0.1.1         1.16.0      A Helm chart for Kubernetes

# Download the helm chart
helm fetch crpocpipelinedevopsazure/my-test-flow

# Unpack the helm chart
tar -xvzf my-test-flow-*.tgz
```

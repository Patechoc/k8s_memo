# Microsoft-Azure/DockerHub (Docker registry) Continuous Delivery

Tutorials:

* [Quickstart: Deploy an Azure Container Service (AKS) cluster](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough)

## Provision Azure with your configuration of the Kubernetes cluster


### Log in to your Azure account


```shell
$ source env.sh
$ az login
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code C7WB8K7VZ to authenticate.

[
  {
    "cloudName": "AzureCloud",
    "id": "12345678-abcd-1a2b3c-",
    "isDefault": true,
    "name": "Pay-As-You-Go",
    "state": "Enabled",
    "tenantId": "987654321-abcdef-1234-edcba",
    "user": {
      "name": "youKKnowMe@outlook.com",
      "type": "user"
    }
  }
]

```


```shell
$ az group list | grep name
    "name": "patrickTest",
...


$
$ az group create --location="${AZURE_REGION}" --name="${AZURE_RESOURCE_GROUP}"
{
  "id": "/subscriptions/96155389-5b91-4c71-a852-9c60175f7354/resourceGroups/patrickTest",
  "location": "westeurope",
  "managedBy": null,
  "name": "patrickTest",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}

$ az group list | grep name
...

```

### Create a Resource Group

```shell
$ az group create --location="${AZURE_REGION}" --name="${AZURE_RESOURCE_GROUP}"
{
  "id": "/subscriptions/96155389-5b91-4c71-a852-9c60175f7354/resourceGroups/patrickTest",
  "location": "westeurope",
  "managedBy": null,
  "name": "patrickTest",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}

```


###  Provision your Resource Group with everything will be set up within (e.g. K8s cluster with AKS)

```shell
$ az aks create     --resource-group="${AZURE_RESOURCE_GROUP}" \
>     --name="${AZURE_AKS}" \
>     --location="${AZURE_REGION}" \
>     --generate-ssh-keys \
>     --node-count=2
Finished service principal creation[##################################]  100.0000%
{
  "id": "/subscriptions/96155389-5b91-4c71-a852-9c60175f7354/resourcegroups/patrickTest/providers/Microsoft.ContainerService/managedClusters/aks-patrickTest",
  "location": "westeurope",
  "name": "aks-patrickTest",
  "properties": {
    "accessProfiles": null,
    "agentPoolProfiles": [
      {
        "count": 2,
        "dnsPrefix": null,
        "fqdn": null,
        "name": "nodepool1",
        "osDiskSizeGb": null,
        "osType": "Linux",
        "ports": null,
        "storageProfile": "ManagedDisks",
        "vmSize": "Standard_D1_v2",
        "vnetSubnetId": null
      }
    ],
    "dnsPrefix": "patr-961553",
    "fqdn": "patr-961553-0df1f8ba.hcp.westeurope.azmk8s.io",
    "kubernetesVersion": "1.7.7",
    "linuxProfile": {
      "adminUsername": "azureuser",
      "ssh": {
        "publicKeys": [
          {
            "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCrHu+nd6LjgfFBVu52rg+rqNAjRW2oTlN0qSkJXSBDdxTMgo7Amjw5yuy2Z2u5kiY00PWzeqmCQLvF3x4w2mYaBEPi47FMZu1S3ijRwz5gVN/eT2QH6j1JKC8o6lz2+c5VGms7MOWM7S589vJGbG8gFCTh4/N1ZmIqAPRmVi3iWp+ZOLIioGJJcT5dZItQIWToArNRG7fPKZqO7KAgKwmMMEfHmiuV9Ci2qMy2O+KlpirxEANIcBcQC+LuXxo7qrT2PdpXk2uQFqFWzanRZRU3IplWa0tVwGhL14g1WlXdtSQJ4si94eyf8OSAzljTvqxvjwXtZUNhpm8UnOOCcgrD patrick.merlot@gmail.com\n"
          }
        ]
      }
    },
    "provisioningState": "Succeeded",
    "servicePrincipalProfile": {
      "clientId": "a70e13db-6d1d-475a-bf75-147623603c31",
      "keyVaultSecretRef": null,
      "secret": null
    }
  },
  "resourceGroup": "patrickTest",
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters"
}
```

### Obtain the (new) credentials for the cluster

> This might overwrite your credentials from previous runs

```shell
$ az aks get-credentials \
     --resource-group="${AZURE_RESOURCE_GROUP}" \
     --name="${AZURE_AKS}"
No Kubernetes access profiles found. Cluster provisioning state is "Succeeded".
```

### Verify that we can talk to the cluster

```shell
kubectl get nodes > /dev/null
```


### Add the credentials of DockerHub to allow it to talk to the cluster

```shell
echo -n "your-DockerHub-username" > ./username.txt
echo -n "your-DockerHub-password" > ./password.txt

kubectl create secret docker-registry ${DOCKERHUB_SECRET_NAME} \
    --docker-server="https://index.docker.io/v1/" \
    --docker-username=patrickmerlot \
    --docker-password=look-in-the-shippable-integration \
    --docker-email=patrickmerlot@gmail.com
```


# Set up the pipeline in the repository

```shell
mkdir -p ${APPLICATION_ID}
emacs shippable.yml
...
```
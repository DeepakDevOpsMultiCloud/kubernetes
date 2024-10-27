# Create an Azure Kubernetes Service (AKS) Cluster

Follow these steps to create an AKS cluster in Azure using a Bash script.

## Prerequisites

- Ensure you have the Azure CLI installed.
- Log in to your Azure account using the command:
  ```bash
  az login
  ```
## Steps to Create AKS Cluster
Set Up Environment Variables: Open your terminal and run the following commands to set up the necessary environment variables:
```bash
export RANDOM_ID="$(openssl rand -hex 3)"
export MY_RESOURCE_GROUP_NAME="myAKSResourceGroup$RANDOM_ID"
export REGION="eastus"
export MY_AKS_CLUSTER_NAME="myAKSCluster$RANDOM_ID"
export MY_DNS_LABEL="ltdevops$RANDOM_ID"

```
## Create a Resource Group: Use the following command to create a new resource group:
```bash
az group create --name $MY_RESOURCE_GROUP_NAME --location $REGION

```
## Create the AKS Cluster: Run the command below to create the AKS cluster:
```bash
az aks create \
    --resource-group $MY_RESOURCE_GROUP_NAME \
    --name $MY_AKS_CLUSTER_NAME \
    --node-count 1 \
    --generate-ssh-keys \
    --node-vm-size 'Standard_B2s'

```
## Get AKS Credentials: Finally, retrieve the credentials for your AKS cluster with this command:
```bash
az aks get-credentials --resource-group $MY_RESOURCE_GROUP_NAME --name $MY_AKS_CLUSTER_NAME
`
```
## Verification
To verify that your nodes are up and running, use:
```bash
kubectl get nodes
```

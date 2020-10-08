# Learn Azure Arc

This project is used to learn and experiment with Azure Arc for Kubernetes.

## Install Prerequisites

As part of this workshop, we need to make sure all of the preview extensions are up to date.

```bash
az extension add --name connectedk8s
az extension add --name k8sconfiguration

az extension update --name aks-preview
az extension update --name connectedk8s
az extension update --name k8sconfiguration
```

## Create the Kubernetes Cluster

```bash
NAME=cdw-kubernetes-20201008
LOCATION=eastus

az group create -n $NAME -l $LOCATION

az aks create \
--resource-group $NAME \
--name $NAME \
--location $LOCATION \
--kubernetes-version 1.18.8 \
--generate-ssh-keys \
--enable-managed-identity


# --enable-addons monitoring
```

## Add the Cluster to Azure Arc

Make sure you have logged into the cluster before executing the final step.

```bash

az aks get-credentials -g $NAME -n $NAME

az connectedk8s connect --resource-group $NAME --name $NAME --location $LOCATION

```

## Add Monitoring for the Arc-enabled Cluster

Make sure you have logged into the cluster before executing the final step.

```bash

az monitor log-analytics workspace create \
--resource-group $NAME \
--workspace-name $NAME-wksp \
--location $LOCATION

az resource list --resource-type Microsoft.OperationalInsights/workspaces -o json

curl -o enable-monitoring.sh -L https://aka.ms/enable-monitoring-bash-script

export azureArcClusterResourceId="/subscriptions/b9c770d1-cde9-4da3-ae40-95ce1a4fac0c/resourceGroups/cdw-kubernetes-20201008/providers/Microsoft.Kubernetes/connectedClusters/cdw-kubernetes-20201008"
export kubeContext="cdw-kubernetes-20201008"
export logAnalyticsWorkspaceResourceId="/subscriptions/b9c770d1-cde9-4da3-ae40-95ce1a4fac0c/resourcegroups/cdw-kubernetes-20201008/providers/microsoft.operationalinsights/workspaces/cdw-kubernetes-20201008-wksp"

bash enable-monitoring.sh --resource-id $azureArcClusterResourceId

```

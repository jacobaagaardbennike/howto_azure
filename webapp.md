# Pipeline
- [ ] Go to the devops board, and create a task named "Azure pipeline for ..."
- [ ] Click new branch and add a branch named "ticketnumber-azure_pipeline"
- [ ] Check-out branch, and add an azure-pipelines.yml file to it
```YML
trigger:
  - main

jobs:
  - job: BuildDockerImage
    displayName: 'Build Docker Image'
    pool:
      vmImage: 'ubuntu-latest'
    steps:
      - task: Docker@2
        inputs:
          containerRegistry: 'CONTAINERREGISTRY'
          repository: 'CONTAINERNAME'
          command: 'buildAndPush'
          Dockerfile: '**/Dockerfile'
          addPipelineData: false
          addBaseImageData: false
```
If the App or Function is going to use a Dev Deployment Slot, use this:
```YML
trigger:
  branches:
    include:
      - dev/*
    exclude:
      - main

jobs:
  - job: BuildDockerImage
    displayName: 'Build Docker Image'
    pool:
      vmImage: 'ubuntu-latest'
    steps:
      - task: Docker@2
        inputs:
          containerRegistry: 'CONTAINERREGISTRY'
          repository: 'CONTAINERNAME'
          command: 'buildAndPush'
          Dockerfile: '**/Dockerfile'
          addPipelineData: false
          addBaseImageData: false
```
- [ ] **Remember to change CONTAINERREGISTRY and CONTAINERNAME**
- [ ] Commit and go to pipelines
- [ ] Click "new pipeline"
- [ ] Click "Azure Repo Git" and select your repo
- [ ] Click "Existing Azure Pipelines YAML file" and select your file


# General
# Login and set Subscription
```powershell
az login
az account set -s SUBSCRIPTIONUUID
```

# If the app service plan is not in the same RG, find it this way:
```powershell
az appservice plan show -n SERVICEPLAN -g PLANRGGROUPNAME --query "id" --out tsv
```

# Create Web App
```powershell
# Create Resource Group
az group create --name RESOURCEGROUPNAME --location westeurope

# Create App and Deployment Slots
# Production
az webapp create --name APPSERVICE --resource-group RESOURCEGROUPNAME --plan SERVICEPLAN --deployment-container-image-name APPSERVICE

# Staging slot (Optional)
az webapp deployment slot create --name APPSERVICE --resource-group RESOURCEGROUPNAME --slot staging

# Dev slot (Optional)
az webapp deployment slot create --name APPSERVICE --resource-group RESOURCEGROUPNAME --slot dev
```

# Create Function App
```powershell
# Create Resource Group
az group create --name RESOURCEGROUPNAME --location westeurope

# Create storage account
az storage account create --name STORAGENAME --location westeurope --resource-group RESOURCEGROUPNAME --sku Standard_LRS

# Create App and Deployment Slots
# Production
az functionapp create --name FUNCTIONAPP --storage-account STORAGENAME --resource-group RESOURCEGROUPNAME --plan SERVICEPLAN --deployment-container-image-name FUNCTIONAPP

# Staging slot (Optional)
az functionapp deployment slot create --name FUNCTIONAPP --resource-group RESOURCEGROUPNAME --slot staging

# Dev slot (Optional)
az functionapp deployment slot create --name FUNCTIONAPP --resource-group RESOURCEGROUPNAME --slot dev
```

# Setup Container Registry Access for your App
- [ ] Requires an Azure AD group that is assigned the ArcPull role under Access Control (IAM) on your container registry.
```powershell
# Production slot
az webapp/functionapp identity assign --resource-group RESOURCEGROUPNAME --n APPSERVICE
az ad group member add --group GROUPID --member-id PRINCIPALID

# Staging slot (Optional)
az webapp/functionapp identity assign --resource-group RESOURCEGROUPNAME --n APPSERVICE --slot staging
az ad group member add --group GROUPID --member-id PRINCIPALID

# Dev slot (Optional)
az webapp/functionapp identity assign --resource-group RESOURCEGROUPNAME --n APPSERVICE --slot dev
az ad group member add --group GROUPID --member-id PRINCIPALID
```

# Setup VNET on your App and deployment slots
- [ ] If the VNET is in a different resource group, it'll require a path similar to:
'/subscriptions/SUBSCRIPTIONUUID/resourceGroups/VNETRESOURCEGROUPNAME/providers/Microsoft.Network/virtualNetworks/VNETNAME'
```powershell
# Production slot
az webapp/functionapp vnet-integration add --resource-group RESOURCEGROUPNAME --name APPSERVICE --vnet VNETINFO --subnet SUBNETNAME

# Test slot (Optional)
az webapp/functionapp vnet-integration add --resource-group RESOURCEGROUPNAME --name APPSERVICE --vnet VNETINFO --subnet SUBNETNAME --slot staging

# Dev slot (Optional)
az webapp/functionapp vnet-integration add --resource-group RESOURCEGROUPNAME --name APPSERVICE --vnet VNETINFO --subnet SUBNETNAME --slot dev
```

# Setup PE for Slots
```powershell
# Production slot
az network private-endpoint create \
--name pe-APPSERVICE \
--resource-group VNETRESOURCEGROUPNAME \
--vnet-name VNETNAME \
--subnet SUBNETNAME \
--connection-name pe-APPSERVICE-GENERATEANUUID \
--private-connection-resource-id /subscriptions/SUBSCRIPTIONUUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Web/sites/APPSERVICE \
--group-id sites

# staging slot (Optional)
az network private-endpoint create \
--name pe-APPSERVICE-staging \
--resource-group VNETRESOURCEGROUPNAME \
--vnet-name VNETNAME \
--subnet SUBNETNAME \
--connection-name pe-APPSERVICE-staging-GENERATEANUUID \
--private-connection-resource-id /subscriptions/SUBSCRIPTIONUUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Web/sites/APPSERVICE \
--group-id sites-staging

# Dev slot (Optional)
az network private-endpoint create \
--name pe-APPSERVICE-dev \
--resource-group VNETRESOURCEGROUPNAME \
--vnet-name VNETNAME \
--subnet SUBNETNAME \
--connection-name pe-APPSERVICE-dev-GENERATEANUUID \
--private-connection-resource-id /subscriptions/SUBSCRIPTIONUUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Web/sites/APPSERVICE \
--group-id sites-dev
```

# Rust container stuff:
```JSON
  {
    "name": "WEBSITES_PORT",
    "value": "8000",
    "slotSetting": false
  },
    {
    "name": "WEBSITES_CONTAINER_START_LIMIT",
    "value": "1200",
    "slotSetting": false
  },
```

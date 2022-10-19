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

# Create App
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

# Setup Container Registry Access for your App
- [ ] Requires an Azure AD group that is assigned the ArcPull role under Access Control (IAM) on your container registry.
```powershell
# Production slot
az webapp identity assign -group RESOURCEGROUPNAME -name APPSERVICE
az ad group member add --group GROUPID --member-id PRINCIPALID

# Test slot (Optional)
az webapp identity assign -group RESOURCEGROUPNAME -name APPSERVICE -slot test
az ad group member add --group GROUPID --member-id PRINCIPALID

# Dev slot (Optional)
az webapp identity assign -group RESOURCEGROUPNAME -name APPSERVICE -s dev
az ad group member add --group GROUPID --member-id PRINCIPALID
```

# Setup VNET on your App and deployment slots
- [ ] If the VNET is in a different resource group, it'll require a path similar to:
'/subscriptions/SUBSCRIPTIONUUID/resourceGroups/VNETRESOURCEGROUPNAME/providers/Microsoft.Network/virtualNetworks/VNETNAME'
```powershell
# Production slot
az webapp vnet-integration add -g RESOURCEGROUPNAME -n APPSERVICE --vnet VNETINFO --subnet SUBNETNAME

# Test slot (Optional)
az webapp vnet-integration add -g RESOURCEGROUPNAME -n APPSERVICE --vnet VNETINFO --subnet SUBNETNAME -s test

# Dev slot (Optional)
az webapp vnet-integration add -g RESOURCEGROUPNAME -n APPSERVICE --vnet VNETINFO --subnet SUBNETNAME -s dev
```

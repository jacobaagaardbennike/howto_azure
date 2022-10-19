# Pipeline
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


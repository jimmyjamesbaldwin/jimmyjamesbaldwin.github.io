---
layout: post
title: "Project: Azure App Service Website IAC"
categories: [general, project]
tags: [general, project, IAC, ARM Templates, docker]
fullview: false
---

_An ARM template, docker-compose, and pipeline solution for building and deploying an Azure App Service website._

### Some background
Some background - I was initially planning on hosting my personal website on an app service site, as it makes running simple containerised application easy and requires very little management. I'm also quite involved with Azure so building everything was straightforward - however, it slipped my mind that when using the free F1 App Service plan you cannot enabled 'Always On' for the container, so the first request after a period of inactivity takes a long time to respond, which is less than ideal for a website and my domain provider wouldn't let me forward to the public endpoint as it took too long to respond...

I did however put together the container, pipelines and ARM templates before I realised this, so I might as well keep them and make them public for anyone else - the project lives on Azure DevOps as I prefer their CI/CD stuff over GitHub Actions:


[Link to Azure DevOps Project](https://dev.azure.com/jimmyjamesbaldwin/_git/jamesbaldwin.co.uk?path=%2FREADME.md)
### jamesbaldwin.co.uk
The final site:
![The end result...](https://i.imgur.com/FV1zWhG.png)

#### Project structure
```
.
├── README.md
├── app-service-deploy-pipeline.yml        # the yaml for the App Service Deployment pipeline
├── arm-template-deploy-pipeline.yml       # the yaml for the ARM Template Deployment pipeline
├── infra
│   ├── azuredeploy.json                   # the ARM template for the App Service Plan, App Service
│   └── deploymentParameters.json          # the ARM template parameters for this project
└── site
    ├── Dockerfile                         # the Dockerfile for building a simple nginx container
    ├── docker-compose.yaml                # a basic docker-compose file for running the container locally
    └── index.html                         # the one-page website
```

#### Pipelines
There exist two pipelines in the project, as follows, for deploying the infrastructure that hosts the app service application, and for building and deploying the containerised web application. Both pipelines are multi-stage yaml pipelines handling both CI & CD with their code store in the repo, rather then 'Release' pipelines in Az DevOps traditionally used for just deployments.
![The two pipelines](https://i.imgur.com/qOeuzmS.png)

##### ARM Template Deployment
The pipeline for deploying the ARM template is reasonably self-explanatory, it includes a single build step for deploying thet template. ARM template parameters are commited in the repo so are not required to be provided at runtime. Note: this step can only be run once, after which it cannot be run again unless the existing infra is torn down, as the azure website names must be unique.
![A screenshot of the ARM template deployment](https://i.imgur.com/XX65XBG.png)


##### App Service Deployment
The pipeline for deploying the App Service includes a couple of steps, one for building the nginx-based docker image and pushing it to DockerHub (authenticated via the 'DockerHub' Service Connection configured for the project, and another for deploying the app service using the AzureRMWebAppDeployment@4 build task (authenticated via a service principal Service Connection with access scoped to a single RG). Once deployed, the initial attempt to resolve the site can take a few seconds.
![A screenshot of the App Service deployment](https://i.imgur.com/Yw5PNFs.png)

### Local Development
To run the project locally, you can launch the docker-compose file by running the following:
```
cd site
docker-compose up -d
```
...then the site will be accessible on localhost:8080

### To Do
If I find time, I _might_ do the following:
* Update the Dockerfile so it doesn't use a huge Ubuntu image for running this small nginx instance
* Update the Dockerfile so it uses a user, as not doing so is bad practise, introduces security concerns, etc.
GitOps uses git as single source of truth to deliver applications and infra.

Gitops is not only abt application deployment it also manages infra. Before Gitops came intopicture there is no mechanism of versioning and auditing

If source code is having proper tracking why your deployment doesnt have proper tracking. 

we will be putting a declarative yaml in a git repository

lets say u deployed and modified something in code we dont have proper tracking 
and deploying to k8s
here gitops came into picture

If source code is having proper tracking why your deployment doesnt have proper tracking. 
we need to place  a declarative yaml in a git repository

If any devops person updated .yaml manifests and any gitops controller like argo will reads the change and deploys the same to k8s cluster

Declarative --> what u see in repo is what u have deployed inside k8s cluster.

Versioned---> Changes are versioned.
Pulled automatically
Reconciation ---> it only allows changes on git whatever the changes which we do to k8s cluster wont work.(security aspect)
advantages: autohealing,security,auto upgrades,auto reconcialiation,versioning

Any Gitops tool the main purpose is to sync b/w the remote vcs and k8s



vcs ---- micro service repo server(connect to git and get the state)
Application controller micro service  --- connect to k8s and get the state

api service ---> micro service ---> for authenticating  ui/cli of argocd

Dex-- sso capability


redis --> for caching why? lets say incase our application controller goes down and again up after sometime it needs info rit so it will immediately take the content from cache.
Application controller is statefulset why?


Argo installation methods:
yaml manifests
helm
operator

## Argo hands-on

CI pipelines implementation
different microservices


Vote ---> python
result ---> nodejs
worker ---> dotnet

redis--> in memory db
pg ---> db
need to implement CI for above

clone below repo
https://github.com/dockersamples/example-voting-app
pre requiste docker deskyop need to be installed if your machine is windows
then 
docker compose up
access the vote app from http://localhost:8080/
access the result app from http://localhost:8081/


dotnet application fetches this info from redis and posts this to postgres and results app will reds the data from postgres.

we will create 1 ci pipeline for voting micro service 
1 for result microservice
1 for worker microservice

Create a project called voting-app in azure devops.

Navigate to repos and click on import repo and provide this url (https://github.com/dockersamples/example-voting-app)

Se the main branch as default branch in azure devops

Navigate to azure portal --> naviage to azure container registry ---> create new 
registry name: saikrishna
rg: abc
Navigate to pipelines and click on create pipeline and mention where is our code present and select repo and under configuration select buld and push an image to azure container registry.


we are using path-based trigger ----whenever any changes made to vote directory or result directory or worker directory it will trigger automatically.

we r using path based triggering for triggering code changes under result directory
trigger: 
  paths:
    include:
      - result/*
changing the naming from votingapp to Resultapp
imageRepository: 'Resultapp'

Ensure the dockerfile path since we r buiding pipeline for result app check whether its considering the same or not.

We are using our own agent for building the pipeline 
https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/linux-agent?view=azure-devops ---> follow this link for setting up agent.


Navigate to project settings and click on agent pools  and add agent

install docker on agent as its a pre requisite.
below commands need to execute on agent
```bash
1. wget https://vstsagentpackage.azureedge.net/agent/4.251.0/vsts-agent-linux-x64-4.251.0.tar.gz
2. tar zxvf ~/Downloads/vsts-agent-linux-x64-4.251.0.tar.gz
3. ls
4. sudo apt update
5. ./config.sh
6. ./run.sh
7. sudo apt install docker.io
8. service docker status
9. whoami
10. sudo usermod -aG docker ubuntu
11. sudo systemctl restart docker
12. logout
13. docker ps
14. cd myagent/
15. ./run.sh
```
sample azure pipeline for worker service
```bash
# Docker
# Build and push an image to Azure Container Registry
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

trigger:
  paths:
    include:
      - worker/*

resources:
- repo: self

variables:
  # Container registry service connection established during pipeline creation
  dockerRegistryServiceConnection: 'ff16b353-46e3-406b-a6f1-832d047f6c95'
  imageRepository: 'workerapp'
  containerRegistry: 'saikrishna.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/worker/Dockerfile'
  tag: '$(Build.BuildId)'

pool:
  name: "azureagent"

stages:
- stage: Build
  displayName: Build 
  jobs:
  - job: Build
    displayName: Build
   
    steps:
    - task: Docker@2
      displayName: Build 
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'build'
        Dockerfile: 'worker/Dockerfile'
        tags: '$(tag)'
- stage: push
  displayName: push 
  jobs:
  - job: push
    displayName: push
   
    steps:
    - task: Docker@2
      displayName: push
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'push'
        tags: '$(tag)'
```
Create k8s cluster in azure 

create a shell script in azure repo where it has to update the image(i.e, under worker,vote,result) in azure repos from acr automatically to latest based on acr.(registryname/reponame:build number)
Gitops will always monitor the git repos since new image is updated to azure repo it will be deployed to k8s cluster.

continuos reconciliation
drift fixed

```bash
login to k8s cluster
install argocd
configure argocd within k8s cluster
write shell script for update the images which is published to acr
```
```bash
az aks get-credentials --resource-group <Resource-Group-Name> --name <AKS-Cluster-Name>
```
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
```bash
kubectl get pods -n argocd
```






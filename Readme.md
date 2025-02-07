```bash
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
```
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
az login
az aks get-credentials --resource-group abc --name saikrishna --overwrite-existing
kubectl get pods
kubectl get deployments --all-namespaces=true
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd
kubectl get pods -n argocd -w
kubectl get pods -n argocd
```
Configure argoCd
Get the password of argocd for integrating it with azure repo
```bash
kubectl get secrets -n argocd

AzureAD+SaikrishnaKakumanu@saikrishna MINGW64 ~
$ kubectl get secrets -n argocd
NAME                          TYPE     DATA   AGE
argocd-initial-admin-secret   Opaque   1      4m46s
argocd-notifications-secret   Opaque   0      5m19s
argocd-redis                  Opaque   1      4m51s
argocd-secret                 Opaque   5      5m19s

```
```bash
kubectl edit secret argocd-initial-admin-secret -n argocd
```
copy the password from above file
since secrets are base 64 encode we need to decrpt that.
```bash
echo <pwsd> | base64 --decode
```
copy the password and dont include % at the time of copying
```bash
kubectl get svc -n argocd
NAME                                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.0.248.225   <none>        7000/TCP,8080/TCP            10m
argocd-dex-server                         ClusterIP   10.0.33.56     <none>        5556/TCP,5557/TCP,5558/TCP   10m
argocd-metrics                            ClusterIP   10.0.117.116   <none>        8082/TCP                     10m
argocd-notifications-controller-metrics   ClusterIP   10.0.169.101   <none>        9001/TCP                     10m
argocd-redis                              ClusterIP   10.0.64.206    <none>        6379/TCP                     10m
argocd-repo-server                        ClusterIP   10.0.250.159   <none>        8081/TCP,8084/TCP            10m
argocd-server                             ClusterIP   10.0.107.190   <none>        80/TCP,443/TCP               10m
argocd-server-metrics                     ClusterIP   10.0.192.11    <none>        8083/TCP                     10m
```
by deafult its in clusterIP and we need to change that to Nodeport
```bash
kubectl edit svc argocd-server  -n argocd #change the type from clusterIP to NodePort
```
navigate to VMSS and click on instances ---> networking ---> inbound rule

access it vai ui through---> https://23.96.57.41:32104/
navigate to settings and click on repositories
under repo url in argocd provide below
```bash
https://BWU6rAdkPTZEqefdakobhqotGhqHauX9w4Pp6tANtt7Ur5leoCHxJQQJ99BBACAAAAAAAAAAAAASAZDOpZZT@dev.azure.com/saikrishna-org/voting-app/_git/voting-app
instead of https://saikrishna-org@dev.azure.com/saikrishna-org/voting-app/_git/voting-app
```
Note whether the connection status is succesful or not.

Once connection is success
navigate to applications ---> new app--->
argocd-poc --> application name
syncying policy --> automatic(#argocd takes 3 mins for identifying the change)
```bash
we need to argocd that which path or directory that argocd need to look for changes in repo in our case its** k8s-specifications**
```

add a new stage under all the worker,voting,result pipelines 
```bash
```
create a shell script for updation of images in k8s-specifications directory in azure repos.

```bash
#!/bin/bash

set -x

# Set the repository URL
REPO_URL="https://<ACCESS-TOKEN>@dev.azure.com/<AZURE-DEVOPS-ORG-NAME>/voting-app/_git/voting-app"

# Clone the git repository into the /tmp directory
git clone "$REPO_URL" /tmp/temp_repo

# Navigate into the cloned repository directory
cd /tmp/temp_repo

# Make changes to the Kubernetes manifest file(s)
# For example, let's say you want to change the image tag in a deployment.yaml file
sed -i "s|image:.*|image: <ACR-REGISTRY-NAME>/$2:$3|g" k8s-specifications/$1-deployment.yaml

# Add the modified files
git add .

# Commit the changes
git commit -m "Update Kubernetes manifest"

# Push the changes back to the repository
git push

# Cleanup: remove the temporary directory
rm -rf /tmp/temp_repo
```

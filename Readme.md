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








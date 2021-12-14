How to Setup and deploy
=======
1. fork code to your git repository

2. clone to the local to build a container image use Dockerfile for customize 

3. build your containerimage `podman build --tag railimage:versiontag -f ./Dockerfile` or Docker or automation trigger to quay.io 

4. push to container image or use your local registry
``` bash
  podman login registry.quay.io
  podman image ls
  podman tag <image ID> <username>/railimage:versioningtag
  podman push <username>/railimage:versioningtag
```

deploy posgresql
# Prepare

## Create a Secret Key for posgresql

Create a database Secret key for user and password to binding a information in the deployment file

```bash
$ kubectl create secret generic db-user --from-literal=username="username"
$ kubectl create secret generic db-user-pass --from-literal=password="password"
```

## Step for prepare a database pod and service

1. apply a postgresql/volumes.yaml config file to create PV and PVC for Database

```bash
$ kubectl apply -f postgresql/volumes.yaml 
```

2. apply a postgresql/svc.yaml config file to create Service for Database

```bash
$ kubectl apply -f svc.yaml
```

3. apply postgres/deploy.yaml config file to create pod of Database service 

```bash
$ kubectl apply -f postgres/deploy.yaml
```

4. Check a pv pvc service and deployments are up all

```bash
$ kubectl get pv,pvc,svc,pod,deployments
```
 
## deploy New Rails Apps
```bash
$ kubectl create secret generic secret-key-base  --from-literal=secret-key-base="key-base"
```
1. go to deploy

``` bash
cd deploy
```

2. Edit a image Tag version in  deploy/rails/deploy.yaml 
``` bash
 
nvim rails/deploy.yaml
```


3. run database migration/seed

$ $ kubectl exec rails-deployment-5f66f99bb9                         \
          -- bash -c 
          'cd ~/app && RAILS_ENV=production bin/rake  db:setup'   

4. and run a rail service for allow Connection from external can access a rail application pod

```bash
$ kubectl create -f rails/svc.yaml
```

5.Confirm a all pod & service run 
```bash
$ kubectl get svc,pod,deployments


### For Update

1. go to deploy

``` bash
cd deploy
```

2. Edit a image Tag version in  deploy/rails/deploy.yaml 
``` bash
 
nvim rails/deploy.yaml
```


3. run database migration/seed

``` bash
$ kubectl exec rails-deployment-5f66f99bb9                         \
          -- bash -c                                            \
          'cd ~/app && RAILS_ENV=production bin/rake db:migrate' 

```

4. and run a rail Service for allow Connection from external can access a Rail Pods
```bash
$ kubectl create -f rails/svc.yaml
```

5.Confirm a all pod & service run 
```bash
$ kubectl get svc,pod,deployments
```

====

Let's say that our app is now running in the pod named rails-deployment-5f66f99bb9

To execute a rake task, for e.g. db:migrate on this pod, we can run the following command.

$ kubectl exec rails-deployment-5f66f99bb9                         \
          -- bash -c                                       \
          'cd ~/app && RAILS_ENV=production bin/rake db:migrate' 
          
Similarly, we can execute db:seed rake task as well.

If we already have an automated flow for deployments on Kubernetes, we can make use of this approach to programmatically or conditionally run any rake task as per the needs.

Why not to use Kubernetes Jobs to solve this?
I faced some issues while using Kubernetes Jobs to run migration and seed rake tasks.

If the rake task returns a non-zero exit code, the Kubernetes job keeps spawning pods until the task command returns a zero exit code.

To get around the issue mentioned above i needed to unnecessarily implement additional custom logic of checking job status and the status of all the spawned pods.

Capturing the command's STDOUT or STDERR was difficult using Kubernetes job.

Some housekeeping was needed such as manually terminating the job if it wasn't successful. If not done, it will fail to create a Kubernetes job with the same name, which is bound to occur when i perform later deployments.

Because of these issues, i choose not to rely on Kubernetes jobs to solve this problem.
# Installation Log

## Environment
- OS: Ubuntu 24.04
- User: geemajor
- Date: 2026-01-13

## Docker Installation
- apt update 
```
sudo apt update
```
- install docker via docker.io
```
sudo apt install -y docker.io
```
- enable & start docker service + verify
```
geemajor@joy:~$ sudo systemctl enable docker
geemajor@joy:~$ docker --version
Docker version 28.2.2, build 28.2.2-0ubuntu1~24.04.1
geemajor@joy:~$ sudo usermod -aG docker $USER
geemajor@joy:~$ newgrp docker
geemajor@joy:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
17eec7bbc9d7: Pull complete
Digest: sha256:d4aaab6242e0cace87e2ec17a2ed3d779d18fbfd03042ea58f2995626396a274
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

## Clone Repos
1. Navigate to the existing OAI folder
> Because we're using Ubuntu on WSL, we have to use `mnt` to locate the windows directory
```
geemajor@joy:~$ cd /mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src
```
1. Clone repos 1: `OPENAIRINTERFACE`
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src$ git clone https://github.com/OPENAIRINTERFACE/openairinterface5g.git

Output:
Cloning into 'openairinterface5g'...
remote: Enumerating objects: 347989, done.
remote: Counting objects: 100% (11116/11116), done.
remote: Compressing objects: 100% (523/523), done.
remote: Total 347989 (delta 10866), reused 10632 (delta 10592), pack-reused 336873 (from 3)
Receiving objects: 100% (347989/347989), 306.77 MiB | 3.59 MiB/s, done.
Resolving deltas: 100% (286138/286138), done.
Updating files: 100% (3963/3963), done.
```
3. Clone repos 2: `it-dep.git`
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src$ git clone https://github.com/o-ran-sc/it-dep.git

Output:
Cloning into 'it-dep'...
remote: Enumerating objects: 9201, done.
remote: Counting objects: 100% (64/64), done.
remote: Compressing objects: 100% (29/29), done.
remote: Total 9201 (delta 50), reused 38 (delta 35), pack-reused 9137 (from 2)
Receiving objects: 100% (9201/9201), 5.15 MiB | 1.68 MiB/s, done.
Resolving deltas: 100% (5575/5575), done.
Updating files: 100% (1137/1137), done.
```

## Checking the Cloned Repo
Go to its directories and use `ls` to see what's inside.
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src$ cd it-dep
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ ls

INFO.yaml     README.md       bin  demos  o-du-high  ric-common  smo-install  tox.ini
LICENSES.txt  RECIPE_EXAMPLE  ci   docs   ric-aux    ric-dep     tools

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ cd bin
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin$ ls

deploy-ric-aux       package-ric-deployment-tools  undeploy-ric-aux       verify-ric-charts
deploy-ric-platform  prepare-common-templates      undeploy-ric-platform  verify-smo-install
```

## Install Kubernetes
kind (Kubernetes IN Docker). A standard O-RAN SC approach and works on laptops

1. Go home
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin$ cd ~
geemajor@joy:~$ pwd
/home/geemajor
```
2. Proceed with tool installation
```
geemajor@joy:~$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    97  100    97    0     0    170      0 --:--:-- --:--:-- --:--:--   171
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
100 6381k  100 6381k    0     0  1655k      0  0:00:03  0:00:03 --:--:-- 2551k
```
3. Install kind
```
geemajor@joy:~$ chmod +x ./kind
geemajor@joy:~$ sudo mv ./kind /usr/local/bin/kind
[sudo] password for geemajor:
geemajor@joy:~$ kind version
kind v0.23.0 go1.21.10 linux/amd64
```
4.  Install kubectl
```
geemajor@joy:~$ sudo snap install kubectl --classic

2026-01-13T20:59:01+07:00 INFO Waiting for automatic snapd restart...
kubectl 1.34.3 from Canonical✓ installed

geemajor@joy:~$ export PATH=$PATH:/snap/bin
geemajor@joy:~$ kubectl version --client

Client Version: v1.34.3
Kustomize Version: v5.7.1
geemajor@joy:~$
```

## Create a Kind Kubernetes cluster
1. Go home
```
geemajor@joy:~$ cd ~
```
2. Create the cluster
```
geemajor@joy:~$ kind create cluster --name oran-ric

Creating cluster "oran-ric" ...
 ✓ Ensuring node image (kindest/node:v1.30.0) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-oran-ric"
You can now use your cluster with:

kubectl cluster-info --context kind-oran-ric

Have a nice day! 👋
```
3. Verify cluster
```
geemajor@joy:~$ kubectl get nodes

NAME                     STATUS   ROLES           AGE   VERSION
oran-ric-control-plane   Ready    control-plane   28m   v1.30.0
```

## Deploy Near-RT RIC Platform
we will use it-dep to install the Near-RT RIC core services into the kubernetes cluster


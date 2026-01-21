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
I will use it-dep to install the Near-RT RIC core services into the kubernetes cluster

1. RIC prepare deploment 
> [!WARNING]
> It hit an error when I try to prepare the RIC documents

```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ bin/prepare-common-templates

+++ dirname bin/prepare-common-templates
++ cd bin
++ pwd
+ ROOT_DIR=/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin
++ helm version -c --short
++ grep -e '^v3'
+ IS_HELM3=v3.19.4+g7cfb6e4
++ ps -x
++ grep 'helm serve'
++ grep -v grep
++ awk '{print $1}'
+ HELM_REPO_PID=
+ '[' -z '' ']'
+ '[' -z v3.19.4+g7cfb6e4 ']'
+ nohup helm servecm --port=8879 --context-path=/charts --storage local
+ command='curl --silent --output /dev/null  http://127.0.0.1:8879/charts'
++ seq 1 5
+ for i in $(seq 1 5)
+ curl --silent --output /dev/null http://127.0.0.1:8879/charts
+ s=7
+ echo 'Error connecting chartmuseum server. Retrying after 5s'
Error connecting chartmuseum server. Retrying after 5s
+ sleep 5
+ for i in $(seq 1 5)
+ curl --silent --output /dev/null http://127.0.0.1:8879/charts
+ s=7
+ echo 'Error connecting chartmuseum server. Retrying after 5s'
Error connecting chartmuseum server. Retrying after 5s
...
```

# REL-J Near-RT RIC Installation Guide.

## Wireshark VPN Setup
1. Install WireGuard Tools
```
$ sudo apt update
$ sudo apt install -y wireguard wireguard-tools
```
2. Check commands exist
```
geemajor@joy:~$ which wg
/usr/bin/wg

geemajor@joy:~$ which wg-quick
/usr/bin/wg-quick
```
3. Check the `.conf` file
```
geemajor@joy:~$ ls /mnt/d/Downloads/joy/
joy.conf
```
4. Make `/etc/wireguard` directory
```
geemajor@joy:~$ sudo mkdir /etc/wireguard
```
5. Verify:
```
geemajor@joy:~$ ls /etc

ImageMagick-6           dbus-1               gss              lsb-release          profile        subuid
LatexMk                 dconf                gtk-2.0          machine-id           profile.d      subuid-
NetworkManager          debconf.conf         gtk-3.0          magic                protocols      sudo.conf
PackageKit              debian_version       host.conf        magic.mime           pulse          sudo_logsrvd.conf
X11                     debuginfod           hostname         manpath.config       python3        sudoers
adduser.conf            default              hosts            matplotlibrc         python3.12     sudoers.d
alternatives            deluser.conf         init             mercurial            rc0.d          supercat
apache2                 depmod.d             init.d           mime.types           rc1.d          sysctl.conf
apparmor                dhcp                 inputrc          mke2fs.conf          rc2.d          sysctl.d
apparmor.d              dhcpcd.conf          iproute2         modprobe.d           rc3.d          systemd
apport                  dictionaries-common  issue            modules              rc4.d          terminfo
apt                     dnsmasq.d            issue.net        modules-load.d       rc5.d          texmf
bash.bashrc             docker               java-21-openjdk  mtab                 rc6.d          timezone
bash_completion         dpkg                 kernel           nanorc               rcS.d          tmpfiles.d
bash_completion.d       e2scrub.conf         landscape        netconfig            resolv.conf    ubuntu-advantage
bindresvport.blacklist  emacs                ld.so.cache      netplan              rmt            ucf.conf
binfmt.d                environment          ld.so.conf       network              rpc            udev
byobu                   environment.d        ld.so.conf.d     networkd-dispatcher  rsyslog.conf   update-manager
ca-certificates         ethertypes           ldap             networks             rsyslog.d      update-motd.d
ca-certificates.conf    fonts                legal            newt                 security       vconsole.conf
cloud                   fstab                libaudit.conf    nftables.conf        selinux        vim
cni                     fuse.conf            libibverbs.d     nsswitch.conf        sensors.d      vtrgb
console-setup           gai.conf             libnl-3          openmpi              sensors3.conf  vulkan
cracklib                gdb                  libpaper.d       opt                  services       wgetrc
credstore               ghostscript          lighttpd         os-release           sgml           wireguard
...
```

6. Move the `.conf` file
```
geemajor@joy:~$ sudo mv /mnt/d/Downloads/joy/joy.conf /etc/wireguard/
```
7. Verify:
```
geemajor@joy:~$ ls /etc/wireguard
joy.conf
```
8. Fix permissions
```
geemajor@joy:~$ sudo chmod 600 /etc/wireguard/joy.conf
```
9.  Start WireGuard
```
geemajor@joy:~$ sudo wg-quick up joy

[#] ip link add joy type wireguard
[#] wg setconf joy /dev/fd/63
[#] ip -4 address add 10.1.0.102/32 dev joy
[#] ip link set mtu 1420 up dev joy
[#] resolvconf -a joy -m 0 -x
[#] ip -4 route add 192.168.8.0/24 dev joy
[#] ip -4 route add 10.1.0.0/24 dev joy
```
10. Check status
```
geemajor@joy:~$ sudo wg
interface: joy
  public key: 3+8L7mQQ6Uvtb8mqikVKXxdVclWol0yuTeqWsnLXUTk=
  private key: (hidden)
  listening port: 40718

peer: caLigu7cD+4nMajYP0rfRyIYlQKppbD9dBq8xilhJmg=
  endpoint: 140.118.162.83:54087
  allowed ips: 10.1.0.0/24, 192.168.8.0/24
  latest handshake: 1 minute, 53 seconds ago
  transfer: 92 B received, 308 B sent
  persistent keepalive: every 25 seconds
  ```
> [!NOTE]
> Succesfully Set Up Wireshark VPN...

## Cluster reconnaissance
1. Check `kubectl`
```
geemajor@joy:~$ kubectl version --client
Client Version: v1.34.3
Kustomize Version: v5.7.1

geemajor@joy:~$ kubectl get nodes
NAME                  STATUS   ROLES           AGE     VERSION
ric-j-control-plane   Ready    control-plane   4d20h   v1.30.0

geemajor@joy:~$ kubectl get namespaces
NAME                 STATUS   AGE
default              Active   4d20h
kube-node-lease      Active   4d20h
kube-public          Active   4d20h
kube-system          Active   4d20h
local-path-storage   Active   4d20h

geemajor@joy:~$ kubectl get pods -A
NAMESPACE            NAME                                          READY   STATUS    RESTARTS      AGE
kube-system          coredns-7db6d8ff4d-9qf6f                      1/1     Running   1 (53m ago)   4d20h
kube-system          coredns-7db6d8ff4d-d9b8k                      1/1     Running   1 (53m ago)   4d20h
kube-system          etcd-ric-j-control-plane                      1/1     Running   0             52m
kube-system          kindnet-dmmv5                                 1/1     Running   3 (53m ago)   4d20h
kube-system          kube-apiserver-ric-j-control-plane            1/1     Running   0             52m
kube-system          kube-controller-manager-ric-j-control-plane   1/1     Running   6 (53m ago)   4d20h
kube-system          kube-proxy-pt7h7                              1/1     Running   1 (53m ago)   4d20h
kube-system          kube-scheduler-ric-j-control-plane            1/1     Running   6 (53m ago)   4d20h
local-path-storage   local-path-provisioner-988d74bc-sckqp         1/1     Running   2 (52m ago)   4d20h
```
2. Local Tools Check
```
geemajor@joy:~$ docker --version
Docker version 28.2.2, build 28.2.2-0ubuntu1~24.04.1

geemajor@joy:~$ helm version
version.BuildInfo{Version:"v3.19.4", GitCommit:"7cfb6e486dac026202556836bb910c37d847793e", GitTreeState:"clean", GoVersion:"go1.24.11"}

geemajor@joy:~$ git --version
git version 2.43.0
```

## RIC Deployment

### 1. Prepare Common Template
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ ./bin/prepare-common-templates

+++ dirname ./bin/prepare-common-templates
++ cd ./bin
++ pwd
+ ROOT_DIR=/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin
++ helm version -c --short
++ grep -e '^v3'
+ IS_HELM3=v3.19.4+g7cfb6e4
++ ps -x
++ grep -v grep
++ grep 'helm serve'
++ awk '{print $1}'
+ HELM_REPO_PID=
+ '[' -z '' ']'
+ '[' -z v3.19.4+g7cfb6e4 ']'
+ nohup helm servecm --port=8879 --context-path=/charts --storage local
+ command='curl --silent --output /dev/null  http://127.0.0.1:8879/charts'
++ seq 1 5
+ for i in $(seq 1 5)
+ curl --silent --output /dev/null http://127.0.0.1:8879/charts
+ s=7
+ echo 'Error connecting chartmuseum server. Retrying after 5s'
Error connecting chartmuseum server. Retrying after 5s
+ sleep 5
+ for i in $(seq 1 5)
+ curl --silent --output /dev/null http://127.0.0.1:8879/charts
+ s=7
+ echo 'Error connecting chartmuseum server. Retrying after 5s'
Error connecting chartmuseum server. Retrying after 5s
```
> [!IMPORTANT]
> If the `Error connecting chartmuseum server` happened, do these steps.

- Fix Chart Museum
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ helm plugin list
NAME    VERSION DESCRIPTION
servecm 0.1.0   Start a ChartMuseum web server

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ helm plugin install https://github.com/chartmuseum/helm-push
Downloading and installing helm-push v0.10.4 ...
https://github.com/chartmuseum/helm-push/releases/download/v0.10.4/helm-push_0.10.4_linux_amd64.tar.gz
Installed plugin: cm-push

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ helm servecm --help
NAME:
   ChartMuseum - Helm Chart Repository with support for Amazon S3, Google Cloud Storage, Oracle Cloud Infrastructure Object Storage and Openstack

USAGE:
   chartmuseum [global options] command [command options] [arguments...]

VERSION:
   0.15.0 (build 460d8ec)

COMMANDS:
   help, h  Shows a list of commands or help for one command
...
```
- Patch `bin/prepare-common-templates` script
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ nano bin/prepare-common-templates
```
When inside, Find `helm servecm`. You will see something like
```
nohup helm servecm --port=8879 --context-path=/charts --storage local &
```
Replace the part with these lines of code
```
if [ -z "$HELM_REPO_PID" ]
then
    if [ -z "$IS_HELM3" ]
    then
      nohup helm serve >& /dev/null &
    else
      nohup helm servecm \
        --port=8879 \
        --context-path=/charts \
        --storage local \
        --storage-local-rootdir /tmp/chartmuseum \
        >& /dev/null &
    fi
fi
```
- Run `bin/prepare-common-templates` again
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ ./bin/prepare-common-templates

+++ dirname ./bin/prepare-common-templates
++ cd ./bin
++ pwd
+ ROOT_DIR=/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin
++ helm version -c --short
++ grep -e '^v3'
+ IS_HELM3=v3.19.4+g7cfb6e4
++ ps -x
++ grep 'helm serve'
++ grep -v grep
++ awk '{print $1}'
+ HELM_REPO_PID=
+ '[' -z '' ']'
+ '[' -z v3.19.4+g7cfb6e4 ']'
+ nohup helm servecm --port=8879 --context-path=/charts --storage local --storage-local-rootdir /tmp/chartmuseum
+ command='curl --silent --output /dev/null  http://127.0.0.1:8879/charts'
++ seq 1 5
+ for i in $(seq 1 5)
+ curl --silent --output /dev/null http://127.0.0.1:8879/charts
+ s=7
+ echo 'Error connecting chartmuseum server. Retrying after 5s'
Error connecting chartmuseum server. Retrying after 5s
+ sleep 5
+ for i in $(seq 1 5)
+ curl --silent --output /dev/null http://127.0.0.1:8879/charts
+ s=0
+ break
+ '[' 0 -gt 0 ']'
+ '[' v3.19.4+g7cfb6e4 ']'
++ grep HELM_REPOSITORY_CACHE
++ helm env
+ eval 'HELM_REPOSITORY_CACHE="/home/geemajor/.cache/helm/repository"'
++ HELM_REPOSITORY_CACHE=/home/geemajor/.cache/helm/repository
+ HELM_LOCAL_REPO=/home/geemajor/.cache/helm/repository/local/
+ mkdir -p /home/geemajor/.cache/helm/repository/local/
++ cat /mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin/../ric-common/Common-Template/helm/ric-common/Chart.yaml
++ grep version
++ awk '{print $2}'
+ COMMON_CHART_VERSION=3.3.2
+ helm package -d /tmp /mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin/../ric-common/Common-Template/helm/ric-common
Successfully packaged chart and saved it to: /tmp/ric-common-3.3.2.tgz
+ cp /tmp/ric-common-3.3.2.tgz /home/geemajor/.cache/helm/repository/local/
++ cat /mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin/../ric-common/Common-Template/helm/aux-common/Chart.yaml
++ grep version
++ awk '{print $2}'
+ AUX_COMMON_CHART_VERSION=3.0.0
+ helm package -d /tmp /mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin/../ric-common/Common-Template/helm/aux-common
Successfully packaged chart and saved it to: /tmp/aux-common-3.0.0.tgz
+ cp /tmp/aux-common-3.0.0.tgz /home/geemajor/.cache/helm/repository/local/
++ cat /mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin/../ric-common/Common-Template/helm/nonrtric-common/Chart.yaml
++ grep version
++ awk '{print $2}'
+ NONRTRIC_COMMON_CHART_VERSION=2.0.0
+ helm package -d /tmp /mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin/../ric-common/Common-Template/helm/nonrtric-common
Successfully packaged chart and saved it to: /tmp/nonrtric-common-2.0.0.tgz
+ cp /tmp/nonrtric-common-2.0.0.tgz /home/geemajor/.cache/helm/repository/local/
+ helm repo index /home/geemajor/.cache/helm/repository/local/
+ helm repo remove local
Error: no repo named "local" found
+ helm repo add local http://127.0.0.1:8879/charts
"local" has been added to your repositories
```
> [!NOTE]
> - ChartMuseum **STARTED SUCCESSFULLY**
> - Helm charts packaged **PERFECTLY**
> - Local Helm repo index **CREATED**

Check with:
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ mkdir -p /tmp/chartmuseum
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ cp ~/.cache/helm/repository/local/*.tgz /tmp/chartmuseum/
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ helm repo index /tmp/chartmuseum
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "localcm" chart repository
Update Complete. ⎈Happy Helming!⎈

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ helm search repo local
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
local/aux-common        3.0.0                           Common templates for inclusion in other charts
local/nonrtric-common   2.0.0                           NONRTRIC Common templates for inclusion in othe...
local/ric-common        3.3.2                           Common templates for inclusion in other charts
localcm/aux-common      3.0.0                           Common templates for inclusion in other charts
localcm/nonrtric-common 2.0.0                           NONRTRIC Common templates for inclusion in othe...
localcm/ric-common      3.3.2                           Common templates for inclusion in other charts
```

> [!IMPORTANT]
> - Bug type: Helm ChartMuseum storage path mismatch
> - Symptom: Repo exists but helm search repo returns nothing
> - Fix: Align packaged chart location with ChartMuseum storage and regenerate index

### 2. Deploy RIC Platform (Near-RT Core)


### 3. Deploy Auxiliary Services

### 4. Verify
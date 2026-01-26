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
> [!NOTE]
> The Near-RT RIC deployment targets O-RAN SC Release J.  
> An E-release–based reference recipe was selected to improve compatibility with modern Kubernetes versions, as Cherry-based recipes rely heavily on deprecated APIs.

#### 2.1 Copy E-Release Recipe Example and Rename
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ cd ric-dep
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/ric-dep$ cp RECIPE_EXAMPLE/example_recipe_oran_e_release.yaml relj-wg11-recipe.yaml
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/ric-dep$ cd ..
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ ./bin/deploy-ric-platform -f ric-dep/relj-wg11-recipe.yaml
```

#### 2.2 Deploy RIC Platform
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ ./bin/deploy-ric-platform -f ric-dep/relj-wg11-recipe.yaml
```
output:
```
+++ dirname /mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin/prepare-common-templates
++ cd /mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin
++ pwd
+ ROOT_DIR=/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/bin
++ helm version -c --short
++ grep -e '^v3'
+ IS_HELM3=v3.19.4+g7cfb6e4
++ grep 'helm serve'
++ ps -x
++ grep -v grep
++ awk '{print $1}'
+ HELM_REPO_PID=8770
+ '[' -z 8770 ']'
+ command='curl --silent --output /dev/null  http://127.0.0.1:8879/charts'
++ seq 1 5
+ for i in $(seq 1 5)
+ curl --silent --output /dev/null http://127.0.0.1:8879/charts
+ s=0
+ break
+ '[' 0 -gt 0 ']'
+ '[' v3.19.4+g7cfb6e4 ']'
++ helm env
++ grep HELM_REPOSITORY_CACHE
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
"local" has been removed from your repositories
+ helm repo add local http://127.0.0.1:8879/charts
"local" has been added to your repositories
Deploying RIC infra components [infrastructure dbaas appmgr rtmgr e2mgr e2term a1mediator submgr vespamgr o1mediator alarmmanager ]
Note that the following optional components are NOT being deployed: {influxdb jaegeradapter}. To deploy them add them with -c to the default component list of the install command
configmap "ricplt-recipe" deleted from ricplt namespace
configmap/ricplt-recipe created
Add cluster roles
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "localcm" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 7 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: [resource mapping not found for name: "kongconsumers.configuration.konghq.com" namespace: "" from "": no matches for kind "CustomResourceDefinition" in version "apiextensions.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "kongcredentials.configuration.konghq.com" namespace: "" from "": no matches for kind "CustomResourceDefinition" in version "apiextensions.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "kongplugins.configuration.konghq.com" namespace: "" from "": no matches for kind "CustomResourceDefinition" in version "apiextensions.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "kongingresses.configuration.konghq.com" namespace: "" from "": no matches for kind "CustomResourceDefinition" in version "apiextensions.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "r4-infrastructure-kong" namespace: "" from "": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "r4-infrastructure-prometheus-alertmanager" namespace: "" from "": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "r4-infrastructure-prometheus-server" namespace: "" from "": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "r4-infrastructure-kong" namespace: "" from "": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "r4-infrastructure-prometheus-alertmanager" namespace: "" from "": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "r4-infrastructure-prometheus-server" namespace: "" from "": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "r4-infrastructure-kong" namespace: "" from "": no matches for kind "Role" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "ricxapp-tiller-base" namespace: "ricxapp" from "": no matches for kind "Role" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "ricxapp-tiller-operation" namespace: "ricinfra" from "": no matches for kind "Role" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "ricxapp-tiller-deployer" namespace: "ricxapp" from "": no matches for kind "Role" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "tiller-secret-creator-xpfsjs-secret-create" namespace: "ricinfra" from "": no matches for kind "Role" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "r4-infrastructure-kong" namespace: "ricplt" from "": no matches for kind "RoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "svcacct-tiller-ricxapp-ricxapp-tiller-base" namespace: "ricxapp" from "": no matches for kind "RoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "svcacct-tiller-ricxapp-ricxapp-tiller-operation" namespace: "ricinfra" from "": no matches for kind "RoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "svcacct-tiller-ricxapp-ricxapp-tiller-deployer" namespace: "ricxapp" from "": no matches for kind "RoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "tiller-secret-creator-xpfsjs-secret-create" namespace: "ricinfra" from "": no matches for kind "RoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first]
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "localcm" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "localcm" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: [resource mapping not found for name: "svcacct-ricplt-appmgr-ricxapp-access" namespace: "" from "": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "svcacct-ricplt-appmgr-ricxapp-getappconfig" namespace: "" from "": no matches for kind "ClusterRole" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "svcacct-ricplt-appmgr-ricxapp-access" namespace: "ricplt" from "": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "svcacct-ricplt-appmgr-ricxapp-getappconfig" namespace: "ricxapp" from "": no matches for kind "ClusterRoleBinding" in version "rbac.authorization.k8s.io/v1beta1"
ensure CRDs are installed first, resource mapping not found for name: "ingress-ricplt-appmgr" namespace: "" from "": no matches for kind "Ingress" in version "networking.k8s.io/v1beta1"
ensure CRDs are installed first]
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "localcm" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "localcm" chart repository
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: resource mapping not found for name: "ingress-ricplt-e2mgr" namespace: "" from "": no matches for kind "Ingress" in version "networking.k8s.io/v1beta1"
ensure CRDs are installed first
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "localcm" chart repository
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "localcm" chart repository
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: resource mapping not found for name: "ingress-ricplt-a1mediator" namespace: "" from "": no matches for kind "Ingress" in version "networking.k8s.io/v1beta1"
ensure CRDs are installed first
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "localcm" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "localcm" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "localcm" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "localcm" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
Error: INSTALLATION FAILED: cannot re-use a name that is still in use
```

#### 2.3 Deployment Verification: Success / Failed / Partially deployed
- How to check?
```
geemajor@joy:~$ kubectl get pods -n ricplt -w
NAME                                              READY   STATUS             RESTARTS        AGE
deployment-ricplt-alarmmanager-674894b75-5dkph    1/1     Running            0               11m
deployment-ricplt-e2term-alpha-5998cb44f6-rjrsn   1/1     Running            1 (5m31s ago)   12m
deployment-ricplt-o1mediator-7555fbd67c-gtbzp     1/1     Running            0               11m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     CrashLoopBackOff   5 (2m21s ago)   13m
deployment-ricplt-submgr-59c6659784-9kwdt         1/1     Running            0               12m
deployment-ricplt-vespamgr-d686664b-nwptg         1/1     Running            0               12m
statefulset-ricplt-dbaas-server-0                 1/1     Running            0               13m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     Running            6 (2m53s ago)   13m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          1/1     Running            6 (3m10s ago)   13m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     Completed          6 (3m56s ago)   14m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     CrashLoopBackOff   6 (14s ago)     14m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     Running            7 (5m13s ago)   19m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          1/1     Running            7 (5m28s ago)   20m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     Completed          7 (6m16s ago)   20m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     CrashLoopBackOff   7 (13s ago)     21m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     Running            8 (5m8s ago)    25m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          1/1     Running            8 (5m13s ago)   26m
```

The following Near-RT RIC core components were successfully deployed and reached a `Running` state:
- DBaaS
- E2 Termination (E2T)
- Subscription Manager (SubMgr)
- VESPA Manager
- O1 Mediator
- Alarm Manager

These components are sufficient to form a **functional Near-RT RIC control plane** for xApp interaction and security analysis.
</br>

The following components failed to deploy fully or exhibited unstable behavior:
- Kong Ingress Controller
- App Manager (AppMgr)
- Routing Manager (RTMgr) — CrashLoopBackOff observed

> [!WARNING]
> Multiple deployment failures were observed due to incompatibilities between RIC Helm charts and the Kubernetes API version.

Identified Root Causes
- Usage of deprecated Kubernetes APIs removed in Kubernetes ≥1.22:
  - `rbac.authorization.k8s.io/v1beta1`
  - `networking.k8s.io/v1beta1`
  - `apiextensions.k8s.io/v1beta1`
- Missing pre-installed Custom Resource Definitions (CRDs) for Kong
- Partial Helm installations leading to release name reuse conflicts



> [!NOTE]
> **Deployment Status: ⚠️ Partially Successful (Near-RT Core Only)**
> 
> The Near-RT RIC deployment targets O-RAN SC Release J.
An E-release–based reference recipe was selected to improve compatibility with modern Kubernetes versions, as Cherry-based recipes rely heavily on deprecated APIs removed in Kubernetes ≥1.22.

### 3. Deploy Auxiliary Services
Auxiliary services in the Near-RT RIC context provide observability, ingress, and lifecycle support for xApps.
For the purpose of WG11 security analysis, only auxiliary components required to observe or influence xApp behavior were considered.

#### 3.1 Scope Decision (Intentional Partial Deployment)
> [!IMPORTANT]
> The following auxiliary components were intentionally not fully deployed due to Kubernetes API incompatibilities and security relevance considerations:
> - Kong Ingress Controller
> - App Manager (AppMgr)

Rationale:
- WG11 threat scenarios focus on xApp trust, authorization, and interaction boundaries, not full production ingress
- Partial deployment allows analysis of implicit trust assumptions and failure modes
- Observed deployment failures themselves constitute security-relevant findings

#### 3.2 Successfully Available Auxiliary Capabilities
The following auxiliary services were **successfully deployed** as part of the RIC platform installation:
| Component     | Function                | Status  |
| ------------- | ----------------------- | ------- |
| VESPA Manager | Metrics & telemetry     | Running |
| Alarm Manager | Fault & event reporting | Running |
| O1 Mediator   | Management interface    | Running |
| DBaaS         | Persistent state        | Running |

Evidence:
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ helm list -n ricplt
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
r4-alarmmanager ricplt          1               2026-01-21 14:03:35.767436442 +0700 WIB deployed        alarmmanager-5.0.0      1.0
r4-dbaas        ricplt          1               2026-01-21 14:01:45.825924815 +0700 WIB deployed        dbaas-2.0.0             1.0
r4-e2term       ricplt          1               2026-01-21 14:02:33.617372346 +0700 WIB deployed        e2term-3.0.0            1.0
r4-o1mediator   ricplt          1               2026-01-21 14:03:19.777131453 +0700 WIB deployed        o1mediator-3.0.0        1.0
r4-rtmgr        ricplt          1               2026-01-21 14:02:11.231799098 +0700 WIB deployed        rtmgr-3.0.0             1.0
r4-submgr       ricplt          1               2026-01-21 14:02:56.776701208 +0700 WIB deployed        submgr-3.0.0            1.0
r4-vespamgr     ricplt          1               2026-01-21 14:03:08.688487007 +0700 WIB deployed        vespamgr-3.0.0          1.0

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl get pods -n ricplt
NAME                                              READY   STATUS    RESTARTS         AGE
deployment-ricplt-alarmmanager-674894b75-5dkph    1/1     Running   0                43m
deployment-ricplt-e2term-alpha-5998cb44f6-rjrsn   1/1     Running   1 (37m ago)      44m
deployment-ricplt-o1mediator-7555fbd67c-gtbzp     1/1     Running   0                43m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     Running   11 (5m18s ago)   44m
deployment-ricplt-submgr-59c6659784-9kwdt         1/1     Running   0                44m
deployment-ricplt-vespamgr-d686664b-nwptg         1/1     Running   0                43m
statefulset-ricplt-dbaas-server-0                 1/1     Running   0                45m

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl get svc -n ricplt
NAME                                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service-ricplt-alarmmanager-http         ClusterIP   10.96.166.69    <none>        8080/TCP                     43m
service-ricplt-alarmmanager-rmr          ClusterIP   10.96.28.151    <none>        4560/TCP,4561/TCP            43m
service-ricplt-dbaas-tcp                 ClusterIP   None            <none>        6379/TCP                     45m
service-ricplt-e2term-prometheus-alpha   ClusterIP   10.96.91.123    <none>        8088/TCP                     44m
service-ricplt-e2term-rmr-alpha          ClusterIP   10.96.118.77    <none>        4561/TCP,38000/TCP           44m
service-ricplt-e2term-sctp-alpha         NodePort    10.96.93.221    <none>        36422:32222/SCTP             44m
service-ricplt-o1mediator-http           ClusterIP   10.96.124.186   <none>        9001/TCP,8080/TCP,3000/TCP   43m
service-ricplt-o1mediator-tcp-netconf    NodePort    10.96.144.179   <none>        830:30830/TCP                43m
service-ricplt-rtmgr-http                ClusterIP   10.96.121.134   <none>        3800/TCP                     44m
service-ricplt-rtmgr-rmr                 ClusterIP   10.96.113.214   <none>        4561/TCP,4560/TCP            44m
service-ricplt-submgr-http               ClusterIP   None            <none>        3800/TCP                     44m
service-ricplt-submgr-rmr                ClusterIP   None            <none>        4560/TCP,4561/TCP            44m
service-ricplt-vespamgr-http             ClusterIP   10.96.194.222   <none>        8080/TCP,9095/TCP            43m
```
#### 3.3 Auxiliary Deployment Observations
> [!WARNING]
> Multiple auxiliary components failed due to deprecated Kubernetes APIs, revealing tight coupling between RIC Helm charts and legacy cluster assumptions.

Observed issues:
- Missing Kong CRDs (`configuration.konghq.com`)
- Deprecated RBAC APIs (`rbac.authorization.k8s.io/v1beta1`)
- Deprecated Ingress APIs (`networking.k8s.io/v1beta1`)

Security implication:
Control-plane availability and ingress reliability may be affected by platform drift, potentially enabling denial-of-service or misconfiguration exploitation.

### 4. Verify
verify what matters for threat modeling

#### 4.1 Control Plane Verification
Verification focused on confirming the presence of critical Near-RT RIC interfaces required for xApp interaction and control-plane communication.

##### 4.1.1 Helm Release State
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ helm list -n ricplt
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
r4-alarmmanager ricplt          1               2026-01-21 14:03:35.767436442 +0700 WIB deployed        alarmmanager-5.0.0      1.0
r4-dbaas        ricplt          1               2026-01-21 14:01:45.825924815 +0700 WIB deployed        dbaas-2.0.0             1.0
r4-e2term       ricplt          1               2026-01-21 14:02:33.617372346 +0700 WIB deployed        e2term-3.0.0            1.0
r4-o1mediator   ricplt          1               2026-01-21 14:03:19.777131453 +0700 WIB deployed        o1mediator-3.0.0        1.0
r4-rtmgr        ricplt          1               2026-01-21 14:02:11.231799098 +0700 WIB deployed        rtmgr-3.0.0             1.0
r4-submgr       ricplt          1               2026-01-21 14:02:56.776701208 +0700 WIB deployed        submgr-3.0.0            1.0
r4-vespamgr     ricplt          1               2026-01-21 14:03:08.688487007 +0700 WIB deployed        vespamgr-3.0.0          1.0
```
- Core RIC components in deployed state
- No FAILED Helm releases

##### 4.1.2 Pod-Level Health Check
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl get pods -n ricplt
NAME                                              READY   STATUS    RESTARTS         AGE
deployment-ricplt-alarmmanager-674894b75-5dkph    1/1     Running   0                43m
deployment-ricplt-e2term-alpha-5998cb44f6-rjrsn   1/1     Running   1 (37m ago)      44m
deployment-ricplt-o1mediator-7555fbd67c-gtbzp     1/1     Running   0                43m
deployment-ricplt-rtmgr-5c9947764c-nrt2w          0/1     Running   11 (5m18s ago)   44m
deployment-ricplt-submgr-59c6659784-9kwdt         1/1     Running   0                44m
deployment-ricplt-vespamgr-d686664b-nwptg         1/1     Running   0                43m
statefulset-ricplt-dbaas-server-0                 1/1     Running   0                45m
```
- Core services (E2Term, SubMgr, DBaaS) in Running state
- RTMgr exhibiting unstable behavior with frequent restarts (intermittent CrashLoopBackOff observed earlier)

> [!WARNING]
> RTMgr exhibiting unstable behavior with frequent restarts (intermittent CrashLoopBackOff observed earlier)

Security note:
Intermittent failure of routing components is treated as a resilience and availability concern, relevant to WG11 threat modeling.

#### 4.2 Interface Exposure Verification

##### 4.2.1 E2 Interface
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl get svc service-ricplt-e2term-sctp-alpha -n ricplt
NAME                               TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)            AGE
service-ricplt-e2term-sctp-alpha   NodePort   10.96.93.221   <none>        36422:32222/SCTP   72m
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$
```
Confirms:
- SCTP exposure
- Near-RT RIC capable of E2 setup interactions over SCTP

##### 4.2.2 Internal Messaging (RMR)
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl get svc -n ricplt | grep rmr
service-ricplt-alarmmanager-rmr          ClusterIP   10.96.28.151    <none>        4560/TCP,4561/TCP            72m
service-ricplt-e2term-rmr-alpha          ClusterIP   10.96.118.77    <none>        4561/TCP,38000/TCP           73m
service-ricplt-rtmgr-rmr                 ClusterIP   10.96.113.214   <none>        4561/TCP,4560/TCP            73m
service-ricplt-submgr-rmr                ClusterIP   None            <none>        4560/TCP,4561/TCP            72m
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$
```
Confirms:
- RMR-based internal communication between RIC components
- Implicit trust between platform services (no service-level authentication enforced by default)

#### 4.3 Log-Based Verification

##### 1. E2 Termination
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl logs deployment/deployment-ricplt-e2term-alpha -n ricplt | tail -n 50
```
key output lines:
```
sigenaddr: error from getaddrinfo: target=service-ricplt-e2mgr-rmr.ricplt:3801
error=(-2) Name or service not known
```
- E2Term is running
- It is trying to resolve the DNS name:
  ```
  service-ricplt-e2mgr-rmr.ricplt
  ```
- Kubernetes DNS cannot resolve it
- That service (E2Mgr) is not deployed

This is expected in many minimal Near-RT RIC deployments. ⚠️ This is not a crash, not fatal, and not misconfiguration of E2Term itself.
</br>

The Important Part of the output is:
```
25/RMR [INFO] sends:
src=service-ricplt-e2term-rmr-alpha.ricplt:38000
target=service-admission-ctrl-xapp-rmr.ricxapp:4560

target=service-ricplt-a1mediator-rmr.ricplt:4562
```
This means:
- RMR stack is initialized
- E2Term is actively attempting message routing
- Multiple RMR targets are configured
- Message routing attempts continue even when some services are missing

##### 2. Subscription Manager
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl logs deployment/deployment-ricplt-submgr -n ricplt | tail -n 50

{"ts":1768983285270,"crit":"INFO","id":"submgr","mdc":{"time":"2026-01-21T08:14:45"},"msg":"restapi: method=GET url=/ric/v1/health/ready"}
{"ts":1768983300255,"crit":"INFO","id":"submgr","mdc":{"time":"2026-01-21T08:15:00"},"msg":"restapi: method=GET url=/ric/v1/health/alive"}
{"ts":1768983300257,"crit":"INFO","id":"submgr","mdc":{"time":"2026-01-21T08:15:00"},"msg":"restapi: method=GET url=/ric/v1/health/ready"}
{"ts":1768983315257,"crit":"INFO","id":"submgr","mdc":{"time":"2026-01-21T08:15:15"},"msg":"restapi: method=GET url=/ric/v1/health/ready"}
```
- SubMgr is:
  - Running
  - Serving HTTP
  - Responding to Kubernetes health probes
- `/alive` → process is running
- `/ready` → service considers itself operational

##### 3. Router Manager
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl logs deployment/deployment-ricplt-rtmgr -n ricplt --previous | tail -n 50

---Successful startup---
Start rtmgr service
Connection to database established!
rmrClient: RMR is ready
Xapp started, listening on: :8080

---Dependency failure---
cannot get xapp data due to:
lookup service-ricplt-appmgr-http on 10.96.0.10:53: server misbehaving

---Controlled Shutdown---
ERROR: Exiting as nbi failed to get the initial startup data
ERROR: Failed to initialize nbi
```

RTMgr logs confirm successful startup, database connectivity, and RMR initialization. However, repeated failures to retrieve xApp metadata from the App Manager service resulted in controlled termination and subsequent restarts.

# Rel-J To Rel-L Update

## REL-L Recon + Prep

### 1. Switch repo to REL-L tag (LOCAL ONLY)
1. Make sure we on `/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep` directory
2. Do
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ git checkout l-release
Updating files: 100% (343/343), done.
M       bin/prepare-common-templates
M       ric-dep
M       smo-install/onap_oom
Note: switching to 'l-release'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at 7c0cc5f Merge "script to install k8s"
```

### 2. Explore Rel-L Layout
1. check the folder
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ ls
INFO.yaml     RECIPE_EXAMPLE                         chartstorage  docs       ranpm       ric-dep           tools
LICENSES.txt  bin                                    ci            nonrtric   ric-aux     ricplt-role.yaml  tox.ini
README.md     chartmuseum_0.16.2_linux_amd64.tar.gz  demos         o-du-high  ric-common  smo-install
```

2. find and make sure the ric-dep are complete
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ find ric-dep -maxdepth 2 -type d
ric-dep
ric-dep/bin
ric-dep/ci
ric-dep/docs
ric-dep/docs/_static
ric-dep/helm
ric-dep/helm/3rdparty
ric-dep/helm/a1mediator
ric-dep/helm/alarmmanager
ric-dep/helm/appmgr
ric-dep/helm/dbaas
ric-dep/helm/e2mgr
ric-dep/helm/e2term
ric-dep/helm/infrastructure
ric-dep/helm/jaegeradapter
ric-dep/helm/o1mediator
ric-dep/helm/redis-cluster
ric-dep/helm/rsm
ric-dep/helm/rtmgr
ric-dep/helm/submgr
ric-dep/helm/vespamgr
ric-dep/helm/xapp-onboarder
ric-dep/RECIPE_EXAMPLE
```

### 3. Find Helm Charts
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ find ric-dep -type d -name "*helm*"
ric-dep/helm

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ find ric-dep -type d -name "*e2*"
ric-dep/helm/e2mgr
ric-dep/helm/e2term
```

### 4. Search Release Docs
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ find . -iname "*release*"
./.releases
./.releases/container-release-it-dep-init.yaml
./.releases/container-release-it-dep-secret.yaml
./docs/release-notes.rst
./ric-dep/docs/release-notes.rst
./ric-dep/RECIPE_EXAMPLE/example_recipe_oran_cherry_release.yaml
./ric-dep/RECIPE_EXAMPLE/example_recipe_oran_dawn_release.yaml
./ric-dep/RECIPE_EXAMPLE/example_recipe_oran_e_release.yaml
./smo-install/multicloud-k8s/releases
./smo-install/onap_oom/docs/sections/guides/deployment_guides/oom_argo_release_deploy.rst
./smo-install/onap_oom/docs/sections/guides/deployment_guides/oom_helm_release_repo_deploy.rst
./smo-install/onap_oom/docs/sections/release_notes
./smo-install/onap_oom/docs/sections/release_notes/release-notes-amsterdam.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-beijing.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-casablanca.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-dublin.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-elalto.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-frankfurt.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-guilin.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-honolulu.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-istanbul.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-jakarta.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-kohn.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-london.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-montreal.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-newdelhi.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes-oslo.rst
./smo-install/onap_oom/docs/sections/release_notes/release-notes.rst

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ ls docs || true
_static       conf.yaml             images                   installation-nonrtric.rst  overview.rst           ric
api-docs.rst  developer-guides.rst  index.rst                installation-ric.rst       release-notes.rst
conf.py       favicon.ico           installation-guides.rst  nrtric                     requirements-docs.txt
```

### 5. Kubernetes Version Expectations
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ grep -R "kubernetes" -n .
```

### 6. Registry Expectations
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ grep -R "nexus3.o-ran-sc.org" -n ric-dep
ric-dep/helm/a1mediator/values.yaml:29:    registry: "nexus3.o-ran-sc.org:10002/o-ran-sc"
ric-dep/helm/alarmmanager/values.yaml:23:    registry: "nexus3.o-ran-sc.org:10002/o-ran-sc" #"ranco-dev-tools.eastus.cloudapp.azure.com:10001"
ric-dep/helm/appmgr/values.yaml:28:          registry: nexus3.o-ran-sc.org:10002/o-ran-sc
ric-dep/helm/appmgr/values.yaml:60:     registry: "nexus3.o-ran-sc.org:10002/o-ran-sc"
ric-dep/helm/appmgr/values.yaml:64:     registry: "nexus3.o-ran-sc.org:10002/o-ran-sc"
ric-dep/helm/dbaas/values.yaml:20:    registry: "nexus3.o-ran-sc.org:10002/o-ran-sc"
ric-dep/helm/e2mgr/values.yaml:28:    registry: "nexus3.o-ran-sc.org:10002/o-ran-sc"
ric-dep/helm/e2term/values.yaml:28:      registry: "nexus3.o-ran-sc.org:10002/o-ran-sc"
...
```

## Fix Rel-J Health Before Touching Rel-L
1. Find Actual Control-Plane IP
```
geemajor@joy:~$ ip route | grep joy
10.1.0.0/24 dev joy scope link
192.168.8.0/24 dev joy scope link
```
2. Get cluster names
```
geemajor@joy:~$ kind get clusters
oran-ric
ric-j
```
3. Start both clusters
```
geemajor@joy:~$ docker start ric-j-control-plane oran-ric-control-plane
ric-j-control-plane
oran-ric-control-plane
```
4. Results
```
geemajor@joy:~$ kubectl get pods -A
NAMESPACE            NAME                                              READY   STATUS    RESTARTS        AGE
kube-system          coredns-7db6d8ff4d-9qf6f                          1/1     Running   3 (2m ago)      10d
kube-system          coredns-7db6d8ff4d-d9b8k                          1/1     Running   3 (119s ago)    10d
kube-system          etcd-ric-j-control-plane                          1/1     Running   2 (2m ago)      6d1h
kube-system          kindnet-dmmv5                                     1/1     Running   10 (2m ago)     10d
kube-system          kube-apiserver-ric-j-control-plane                1/1     Running   7 (2m ago)      6d1h
kube-system          kube-controller-manager-ric-j-control-plane       1/1     Running   50 (2m ago)     10d
kube-system          kube-proxy-pt7h7                                  1/1     Running   3 (119s ago)    10d
kube-system          kube-scheduler-ric-j-control-plane                1/1     Running   49 (119s ago)   10d
local-path-storage   local-path-provisioner-988d74bc-sckqp             1/1     Running   6 (45s ago)     10d
ricplt               deployment-ricplt-alarmmanager-674894b75-5dkph    1/1     Running   2 (119s ago)    5d4h
ricplt               deployment-ricplt-e2term-alpha-5998cb44f6-rjrsn   0/1     Running   21 (2m ago)     5d4h
ricplt               deployment-ricplt-o1mediator-7555fbd67c-gtbzp     1/1     Running   2 (2m ago)      5d4h
ricplt               deployment-ricplt-rtmgr-5c9947764c-nrt2w          1/1     Running   154 (2m ago)    5d4h
ricplt               deployment-ricplt-submgr-59c6659784-9kwdt         1/1     Running   15 (2m ago)     5d4h
ricplt               deployment-ricplt-vespamgr-d686664b-nwptg         1/1     Running   2 (2m ago)      5d4h
ricplt               statefulset-ricplt-dbaas-server-0                 1/1     Running   8 (41s ago)     5d4h
```

## Deploy Rel-L Near-RT RIC Platform

### 1. Rel-L Prep
1. Pick Cluster
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl config get-contexts
CURRENT   NAME            CLUSTER         AUTHINFO        NAMESPACE
          kind-oran-ric   kind-oran-ric   kind-oran-ric
*         kind-ric-j      kind-ric-j      kind-ric-j
```
we switch to `kind-oran-ric`
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl config use-context kind-oran-ric
Switched to context "kind-oran-ric".
```


2. StorageClass Check
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl get sc
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  11d
```
This satisfies Near-RT RIC PVC requirements.

3. Chartmuseum check
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ ls chartmuseum*
chartmuseum_0.16.2_linux_amd64.tar.gz
```
present


### 2. Deployment Pipeline

#### 1. Start ChartMuseum

1. Download chartmuseum
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ curl -L -o chartmuseum-v0.16.2-linux-amd64.tar.gz \
https://get.helm.sh/chartmuseum-v0.16.2-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 18.6M  100 18.6M    0     0  2096k      0  0:00:09  0:00:09 --:--:-- 2190k
```

2. Check file tipe
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ file chartmuseum-v0.16.2-linux-amd64.tar.gz
chartmuseum-v0.16.2-linux-amd64.tar.gz: gzip compressed data, from Unix, original size modulo 2^32 68730880
```

3. extract
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ tar -xzf chartmuseum-v0.16.2-linux-amd64.tar.gz
```

4. verify
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ ls
INFO.yaml       chartmuseum-v0.16.2-linux-amd64.tar.gz  linux-amd64  ric-common        tox.ini
LICENSES.txt    chartstorage                            nonrtric     ric-dep
README.md       ci                                      o-du-high    ricplt-role.yaml
RECIPE_EXAMPLE  demos                                   ranpm        smo-install
bin             docs                                    ric-aux      tools
```

5. go to `linux-amd64` directory, and execute the chartmuseum, and move it to PATH
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ cd linux-amd64

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/linux-amd64$ ls
LICENSE  README.md  chartmuseum

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/linux-amd64$ chmod +x chartmuseum

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/linux-amd64$ ./chartmuseum --version
ChartMuseum version 0.16.2 (build 8795e99)

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/linux-amd64$ sudo mv chartmuseum /usr/local/bin
[sudo] password for geemajor:

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep/linux-amd64$ chartmuseum --version
ChartMuseum version 0.16.2 (build 8795e99)
```

#### 2. Helm repo add
1. Start ChartMuseum
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ mkdir -p chartstorage

chartmuseum \
  --port=8080 \
  --storage=local \
  --storage-local-rootdir=./chartstorage &
[1] 135033
```
2.  Helm repo add
```
geemajor@joy:~$ helm repo add localric http://localhost:8080//localhost:8080
helm repo update
helm repo list
"localric" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Unable to get an update from the "local" chart repository (http://127.0.0.1:8879/charts):
        Get "http://127.0.0.1:8879/charts/index.yaml": dial tcp 127.0.0.1:8879: connect: connection refused
...Unable to get an update from the "localcm" chart repository (http://127.0.0.1:8879/charts):
        Get "http://127.0.0.1:8879/charts/index.yaml": dial tcp 127.0.0.1:8879: connect: connection refused
...Successfully got an update from the "localric" chart repository
Update Complete. ⎈Happy Helming!⎈
NAME            URL
localcm         http://127.0.0.1:8879/charts
local           http://127.0.0.1:8879/charts
localric        http://localhost:8080
```
3. Helm health check
```
geemajor@joy:~$ curl http://localhost:8080/health
{"healthy":true}
```
4. search repo
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ helm search repo localric
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION                                     
localric/a1controller                   2.0.0           2.0.0           A Helm chart for nonrtric a1controller          
localric/a1simulator                    2.1.0           2.0.0           A Helm chart for A1 simulator                   
localric/capifcore                      1.0.0           2.0.0           A Helm chart for CAPIF core                     
localric/common                         1.0.0           1.16.0          A Helm chart third party components             
localric/controlpanel                   2.0.0           2.0.0           A Helm chart for nonrtric controlpanel          
localric/dmaapadapterservice            1.0.0           1.0.0           A Helm chart for Dmaap Adapter Service          
localric/informationservice             1.0.0           1.0.0           A Helm chart for Information Coordinator Service
localric/kong                           1.0.0           1.0.0           A Helm chart for deploying DB-mode Kong with Po...
localric/nonrtric                       1.0.0           test            Open Radio Access Network (ORAN)                
localric/nonrtric-common                2.0.0                           NONRTRIC Common templates for inclusion in othe...
localric/nonrtricgateway                1.0.0           0.0.1           A Helm chart for Nonrtric Gateway               
localric/policymanagementservice        2.0.0           2.0.0           A Helm chart for Policy Management Service      
localric/ranpm                          1.0.0           1.16.0          A Helm chart for RANPM Components               
localric/rappmanager                    1.0.0           2.0.0           A Helm chart for rAppmanager                    
localric/servicemanager                 1.0.0           2.0.0           A Helm chart for ServiceManager                 
localric/smo                            1.0.0           test            Open Radio Access Network (ORAN)                
localric/smo-common                     1.0.0                           SMO Common templates for inclusion in other charts
localric/topology                       1.0.0           1.0.0           A Helm chart to deploy topology                 
localric/topology-exposure-inventory    1.0.0           1.16.0          A Helm chart for Kubernetes                     
```

#### 3. Verify Nexus Registry Reachable
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ curl -I https://nexus3.o-ran-sc.org
HTTP/1.1 200 OK
Server: nginx/1.24.0
Date: Mon, 26 Jan 2026 16:11:36 GMT
Content-Type: text/html
Content-Length: 10267
Connection: keep-alive
Keep-Alive: timeout=5
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Last-Modified: Mon, 26 Jan 2026 16:11:36 GMT
Pragma: no-cache
Cache-Control: no-cache, no-store, max-age=0, must-revalidate, post-check=0, pre-check=0
Expires: 0
Referrer-Policy: no-referrer
Permissions-Policy: camera=(), geolocation=(), microphone=()
Content-Security-Policy: default-src 'none'; script-src 'self' 'unsafe-inline' 'unsafe-eval' www.google-analytics.com; style-src 'self' 'unsafe-inline' fonts.googleapis.com; img-src 'self' https: data:; connect-src 'self' 'unsafe-inline' www.google-analytics.com stats.g.doubleclick.net; frame-src 'self'; font-src 'self' fonts.gstatic.com
Strict-Transport-Security: max-age=15552000
```
>[!Note]
registry connectivity is good

#### 4. Namespace prep
```
geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl create namespace nonrtric
kubectl create namespace smo
kubectl create namespace ranpm
kubectl create namespace ricinfra
namespace/nonrtric created
namespace/smo created
namespace/ranpm created
namespace/ricinfra created

geemajor@joy:/mnt/d/Documents/GitHub/intern_repo/o-ran/OAI/src/it-dep$ kubectl get ns
NAME                 STATUS   AGE
default              Active   13d
kube-node-lease      Active   13d
kube-public          Active   13d
kube-system          Active   13d
local-path-storage   Active   13d
nonrtric             Active   13s
ranpm                Active   10s
ricinfra             Active   8s
smo                  Active   12s
```
>[!Note]
> We're good to proceed

### 3. Deploy Non-RT RIC Umbrella Chart (Rel-L)

# Deploy Full E2E O-RAN Devices

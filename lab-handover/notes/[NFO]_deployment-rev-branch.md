# NFO Deployment Progress — Joy
**Date:** May 19, 2026  
**Task:** Duplicate Nino's NFO deployment (Contribution 1, sub-task NFO)  
**Reference:** `branch: revised` → `nino-o2-nfo/Example.md`

---

## Server Access Status

| Server | Address | Status | Notes |
|--------|---------|--------|-------|
| Galileo | 192.168.8.35 | ✅ SSH OK | StarlingX O-Cloud, `oran-o2` O2 API running |
| Archimede | 192.168.8.69 | ✅ SSH OK | Full SMO platform, my main workspace for NFO |
| Joule | 192.168.206.82 | ❌ SSH Failed | `Permission denied (publickey,password)` even from Galileo root |

**Joule note:** Leo said access is via Galileo root only, but still getting permission denied. Might need SSH key setup (haven't resolved this yet).

---

## What I Found on the Servers

### Galileo (192.168.8.35)
- It's actually a **StarlingX Kubernetes cluster**, not just a jump server
- `oran-o2 / o2api` pod is **running** → O2 IMS interface is up ✅
- `johnson-ns / oai-cu` → **CrashLoopBackOff** → this is the broken gNB (known issue per Leo)
- Leo's files at `/root/leo/`: `bmw`, `nino`, `ocloud-tester-sideload`
- No `Example.md` found locally on Galileo

### Archimede (192.168.8.69)
- Full SMO stack running (ONAP, NonRTRIC, SMO, rApp, etc.) — ~110 pods
- **No NFO or FOCOM pods running** when I first checked
- Leo's files at `/home/ubuntu/leo/`: `focom`, `smo-o2`, `smo-o2-focom`, `rapp-cicd-ocloud`, `cicd-rapp-client`
- No `Example.md` found locally — only exists on GitHub `revised` branch

---

## What I Did

### 1. Cloned NFO revised branch
```bash
cd /home/ubuntu/leo
git clone -b revised https://github.com/bmw-ece-ntust/nino-o2-nfo
```
Confirmed `Example.md` is there and read through all 6 steps.

### 2. Checked NFO deployment files
- Helm chart at `nino-o2-nfo/nfo/k8s/chart/`
- Image: `bmw.ece.ntust.edu.tw/shuchu11/nfo:latest` (Leo's pre-built image)
- Confirmed image is pullable ✅
```bash
docker pull bmw.ece.ntust.edu.tw/shuchu11/nfo:latest
# Status: Image is up to date
```

### 3. Deployed NFO via Helm
```bash
cd /home/ubuntu/leo/nino-o2-nfo/nfo/k8s
helm install nfo ./chart -n o2
# STATUS: deployed, REVISION: 1
```

---

## Current Blockers

### 🔴 Blocker 1: Node at 110/110 pod capacity

NFO pod is stuck `Pending`. First suspected pod count, then confirmed:

```
kubectl get pods -A | grep Running | wc -l
→ 110  (hard limit on single-node cluster)
```

The node `zhongkui` only has 1 node with default `maxPods: 110`. The entire SMO/ONAP/NonRTRIC platform is already filling all 110 slots with legitimate running services.

**What I did to try to fix it:**
- Found ~49 zombie `onap-dcae-ves-collector` pods with `ContainerStatusUnknown` status
- Deleted them to free up slots:
```bash
kubectl delete pods -n onap --field-selector=status.phase=Failed
# 49 pods deleted
```

**But NFO is still Pending**, the 110 slots filled back up with legitimate pods from other namespaces (nonrtric: 26, onap: 24, smo: 23, etc.)

### 🔴 Blocker 2: PVC also stuck Pending

```
kubectl get pvc -n o2
→ nfo    Pending    openebs-hostpath
```

This is a chicken-and-egg problem lol. PVC won't bind until pod is scheduled, pod won't schedule because node is full.

### 🔴 Blocker 3: Joule SSH not accessible

Can't reach `root@192.168.206.82` even from Galileo root. Need this for StarlingX kubeconfig to complete Step 2 of the 6-step NFO flow.

---

## NFO 6-Step Flow Status (from Example.md)

| Step | Description | Status |
|------|-------------|--------|
| Pre | Deploy NFO service itself | 🔴 Blocked — pod pending, node full |
| 1 | Register Cluster | ⏳ Waiting for NFO to be up |
| 2 | Set Credentials (needs StarlingX kubeconfig) | ⏳ Also needs Joule access |
| 3 | Test Connection | ⏳ |
| 4 | Create VNF Descriptor | ⏳ |
| 5 | Create Deployment | ⏳ |
| 6 | Instantiate (deploy gNB) | ⏳ |

---

## Questions for Leo / Prof Ray

1. **Archimede node is at 110/110 pod capacity**, NFO can't be scheduled. Options I see: increase `maxPods` on kubelet, or scale down something temporarily. What's the right move?
2. **Joule SSH** — `Permission denied (publickey,password)` from Galileo root. Is there an SSH key we need to copy first?


---

## Next Steps (pending guidance)

- Wait for Leo/Prof response on pod capacity issue
- Once NFO pod is Running, proceed with 6 curl steps from `Example.md`
- Resolve Joule SSH access 
- Coordinate with Poggy on FOCOM side

---

*Joy report | Based on work done May 19, 2026*
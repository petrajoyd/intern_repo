# Jenkins CI/CD Installation Note
**VM 1 — Jenkins Server**

> Author: Petrajoy Davidson | Date: 2026-04-27 | Status: 🔄 In Progress

---

## Job Checklist

| Phase | Status |
|-------|--------|
| Phase 0: VM Specs | ✅ Done |
| Phase 1: Install Prerequisites | ✅ Done |
| Phase 2: Install Jenkins | ✅ Done |
| Phase 3:  Configure Jenkins Agent | ✅ Done |
| Phase 4:  Install Required Plugins | ✅ Done |
| Phase 5: Set Up Credentials | ✅ Done |
| Phase 6: Create Pipeline Job | ✅ Done |
| Phase 7: Run Pipeline | 🔴 Blocked |

---

## Phase 0: VM Specifications

Record your VM specs before starting anything.

### Commands
```bash
nproc          # CPU cores
free -h        # RAM
df -h          # Disk
cat /etc/os-release  # OS
ip route show  # Correct IP address
```

> ⚠️ **Finding**: Use `ip route show` instead of `ip a` to get the correct routing IP. `ip a` may show a different IP than what's actually routable.

### Results

| Spec | Value |
|------|-------|
| OS | Red Hat Enterprise Linux 9.4 (Plow) |
| CPU Cores | 8 cores |
| RAM | 7.5 GiB total / 6.0 GiB available |
| Swap | 5.0 GiB |
| Disk (root) | 44 GiB total / 39 GiB free |
| Disk (boot) | 960 MiB total / 665 MiB free |
| IP Address | 192.168.8.54 (ens192) |
| Hostname | localhost |
| Username | joy2 |

### Checklist
- [x] CPU cores, RAM, disk, OS recorded
- [x] VM IP address confirmed via `ip route show`

---

## Phase 1: Install Prerequisites

>[!WARNING]
>  This VM runs **RHEL 9**, not Ubuntu. Use `dnf` instead of `apt`. All commands below assume you are running as `root`, no `sudo` needed.


### Step 1: Update System
```bash
dnf update -y
```

### Step 2: Install Java
```bash
dnf install -y java-17-openjdk
java -version
```

>[!WARNING]
> Java 17 was installed here, but Jenkins 2.555.1 requires Java 21 minimum. We had to upgrade later. See Phase 2 troubleshooting.

### Step 3: Install Git
```bash
dnf install -y git
git --version
```

### Step 4: Install Docker
```bash
dnf install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
dnf install -y docker-ce docker-ce-cli containerd.io
systemctl enable --now docker
docker ps
```

### Versions Installed

| Package | Version |
|---------|---------|
| Java | OpenJDK 21.0.11 (upgraded from 17, see Phase 2) |
| Git | 2.47.3 |
| Docker | 29.4.1 |
| containerd | 2.2.3 |

### Checklist
- [x] `dnf update -y`
- [x] `dnf install -y java-17-openjdk` *(upgraded to 21 later)*
- [x] `dnf install -y git`
- [x] Docker installed and running

---

## Phase 2: Install Jenkins

### Step 1: Add Jenkins Repository
```bash
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

### Step 2: Install Jenkins
```bash
dnf install -y jenkins
```

This installs Jenkins 2.555.1.

### Step 3: Start Jenkins
```bash
systemctl enable --now jenkins
systemctl status jenkins
```

>[!WARNING]
> 🔴 **Issue**: Jenkins failed to start with `exit-code 1`.
>
> **Diagnosis**: Running `/usr/bin/jenkins` manually revealed:
> ```
> Running with Java 17 which is older than the minimum required version (Java 21).
> Supported Java versions are: [21, 25]
> ```
>
> **Fix**: Install Java 21 and set it as default:
> ```bash
> dnf install -y java-21-openjdk
> alternatives --config java   # select Java 21
> java -version                # verify: openjdk 21.0.11
> ```
> 🔴 **Issue**: Jenkins still failed after Java 21 install, systemd was still pointing to Java 17.
>
> **Fix**: Create a systemd override:
> ```bash
> mkdir -p /etc/systemd/system/jenkins.service.d/
> cat > /etc/systemd/system/jenkins.service.d/override.conf << EOF
> [Service]
> Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk-21.0.11.0.10-2.el9.x86_64"
> EOF
> systemctl daemon-reload
> ```
> 🔴 **Issue**: Jenkins stuck in failed state after too many restart attempts.
>
> **Fix**:
> ```bash
> systemctl reset-failed jenkins
> systemctl start jenkins
> ```

### Step 4: Verify Jenkins
```bash
systemctl status jenkins
curl http://localhost:8080
```

### Step 5: Access Jenkins UI

>[!WARNING]
> 🔴 **Issue**: `curl http://192.168.8.57:8080` returned "No route to host", wrong IP was used.
>
> **Fix**: Run `ip route show` to get correct IP → `192.168.8.54`

Open browser → `http://192.168.8.54:8080`

Unlock with:
```bash
cat /var/lib/jenkins/secrets/initialAdminPassword
```

Complete setup wizard and install suggested plugins.

### Jenkins Info

| Item | Value |
|------|-------|
| Version | 2.555.1 |
| URL | http://192.168.8.54:8080 |
| Admin User | joy_jenkins |

### Checklist
- [x] Jenkins repo added
- [x] GPG key imported
- [x] `dnf install -y jenkins` → v2.555.1
- [x] Java 21 configured via systemd override
- [x] Jenkins started successfully
- [x] UI accessible at `http://192.168.8.54:8080`
- [x] Setup wizard completed
- [x] Suggested plugins installed

---

## Phase 3: Configure Jenkins Agent

The pipeline script requires an agent labeled `jenkins-agent-01`. Since we only have one VM, we use the built-in node.

### Steps
1. Go to **Manage Jenkins → Nodes → Built-In Node → Configure**
2. Add label: `jenkins-agent-01`
3. Click Save

> ⚠️ **Finding on Nino's doc**: The doc assumes a separate agent node `jenkins-agent-01` already exists but doesn't explain how to create it. For single-VM setups, adding the label to the built-in node works fine.

### Checklist
- [x] Label `jenkins-agent-01` added to built-in node

---

## Phase 4: Install Required Jenkins Plugins

All required plugins were already installed by the suggested plugins wizard. No manual installation needed.

### Verified Plugins

| Plugin | Status |
|--------|--------|
| git | ✅ |
| pipeline | ✅ |
| ssh-credentials | ✅ |
| workflow-job | ✅ |
| github | ✅ |
| credentials-binding | ✅ |

### Checklist
- [x] All required plugins verified ✅

---

## Phase 5: Set Up Credentials

### Credential 1: GitHub Token (`gh_token`)

Generate a GitHub Personal Access Token with scopes: `repo`, `workflow`.

In Jenkins UI: **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

```
Kind:        Username with password   ← IMPORTANT: must be this type
Scope:       Global
Username:    <github_username>
Password:    <personal_access_token>
ID:          gh_token
Description: GitHub Token
```

>[!WARNING]
> 🔴 **Issue**: First attempt used **"GitHub App"** credential type instead of **"Username with password"**. This caused:
> ```
> ERROR: Could not find credentials entry with ID 'gh_token'
> ```
> Even though the ID was correct, the wrong type makes it invisible to the pipeline.
>
> **Fix**: Delete and recreate with correct type: `Username with password`
> ⚠️ **Finding on Nino's doc**: The doc does not specify the credential **Kind**. It should explicitly state `Username with password` to avoid this confusion.

### Credential 2: SSH Private Key (`ssh-pubkey-kenny-192.168.8.53`)

Generate SSH key pair on Jenkins VM:
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/jenkins_key -N ""
```

Copy public key to newton:
```bash
ssh-copy-id -i ~/.ssh/jenkins_key.pub kenny@192.168.8.53
# Password: bmwlab
```

Verify SSH works:
```bash
ssh -i ~/.ssh/jenkins_key kenny@192.168.8.53
```

Copy private key to local PC for upload:
```cmd
# On Windows laptop:
scp joy2@192.168.8.54:/home/joy2/jenkins_key %USERPROFILE%\Desktop\jenkins_key
```

In Jenkins UI: **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

```
Kind:        Secret file
Scope:       Global
File:        jenkins_key (private key file)
ID:          ssh-pubkey-kenny-192.168.8.53
Description: SSH key for newton
```

### Checklist
- [x] GitHub token created with correct type ✅
- [x] SSH key generated and added to newton ✅
- [x] SSH key uploaded to Jenkins credentials ✅

---

## Phase 6: Create Pipeline Job

In Jenkins UI: **New Item → Deploy_KPIMON_xApp → Pipeline → OK**

Paste the Groovy script from Nino's doc Section 3.2 into the Pipeline Script box.

Default parameters verified:

| Parameter | Value |
|-----------|-------|
| REMOTE_HOST | 192.168.8.53 |
| REMOTE_USER | kenny |
| GIT_CREDENTIAL_ID | gh_token |
| GIT_REPO_PATH | github.com/bmw-ece-ntust/kpimon-go-xapp.git |
| GIT_BRANCH | CICD |
| IMAGE_NAME | ric-app-kpimon-go |
| TAG | 1.0.1 |
| CHART_VERSION | 2.0.1 |

### Nino's Pasted Script

```groovy
pipeline {
    agent { label 'jenkins-agent-01' }

    parameters {
        string(name: 'REMOTE_HOST', defaultValue: '192.168.8.53', description: 'Remote IP')
        string(name: 'REMOTE_USER', defaultValue: 'kenny', description: 'SSH Username')
        string(name: 'REMOTE_PASS', defaultValue: 'bmwlab', description: 'SSH Password')
        string(name: 'GIT_CREDENTIAL_ID', defaultValue: 'gh_token', description: 'GitHub Token Credential ID')
        
        string(name: 'GIT_REPO_PATH', defaultValue: 'github.com/bmw-ece-ntust/kpimon-go-xapp.git', description: 'Git Repo')
        string(name: 'GIT_BRANCH', defaultValue: 'CICD', description: 'Branch')
        string(name: 'IMAGE_NAME', defaultValue: 'ric-app-kpimon-go', description: 'Image Name')
        string(name: 'TAG', defaultValue: '1.0.1', description: 'Tag')
        string(name: 'CHART_VERSION', defaultValue: '2.0.1', description: 'Chart Version')
    }

    stages {
       stage('Deploy via SSH Script') {
        steps {
            script {
                // file() retrieves the path to the temporary file Jenkins creates for the secret
                withCredentials([
                    usernamePassword(credentialsId: params.GIT_CREDENTIAL_ID, usernameVariable: 'GH_USER', passwordVariable: 'GH_TOKEN'),
                    file(credentialsId: 'ssh-pubkey-kenny-192.168.8.53', variable: 'SSH_KEY_FILE') 
                ]) {
                    
                    // 1. Define the deployment payload
                    def scriptContent = """#!/bin/bash
                        set -ex
                        export PATH=\$PATH:/usr/local/bin:/usr/bin:/bin
                        
                        echo "--- Target Environment: \$(hostname) ---"
                        mkdir -p ~/CICD_test
                        cd ~/CICD_test
    
                        echo "--- Repository Management ---"
                        rm -rf kpimon-go-xapp
                        git clone https://${GH_TOKEN}@${params.GIT_REPO_PATH} -b ${params.GIT_BRANCH}
    
                        cd kpimon-go-xapp/
                        
                        echo "--- Build Process ---"
                        docker build -t ${params.IMAGE_NAME}:${params.TAG} .
    
                        echo "--- RIC xApp Onboarding ---"
                        if [ -d "deploy" ]; then
                            cd deploy
                            dms_cli onboard config.json schema.json
                            dms_cli install kpimon-go ${params.CHART_VERSION} ricxapp
                        else
                            echo "Error: deploy folder not found!"
                            exit 1
                        fi
                    """.stripIndent()
    
                    // 2. Write the payload script to the local workspace
                    writeFile file: 'deploy_xapp.sh', text: scriptContent
    
                    try {
                        // 3. Execute using the SSH_KEY_FILE path provided by Jenkins
                        sh """
                            # SSH requires the key file to have strict permissions (read-only for owner)
                            chmod 600 ${SSH_KEY_FILE}
                            
                            # Transfer the script
                            scp -i ${SSH_KEY_FILE} -o StrictHostKeyChecking=no deploy_xapp.sh ${params.REMOTE_USER}@${params.REMOTE_HOST}:~/deploy_xapp.sh
                            
                            # Run the script
                            ssh -i ${SSH_KEY_FILE} -tt -o StrictHostKeyChecking=no ${params.REMOTE_USER}@${params.REMOTE_HOST} 'bash ~/deploy_xapp.sh'
                        """
                    } catch (Exception e) {
                        echo "Deployment failed: ${e.message}"
                        error("SSH Deployment Stage Failed")
                    } finally {
                        // 4. Clean up the script; Jenkins automatically cleans up the SSH_KEY_FILE
                        sh "rm -f deploy_xapp.sh"
                    }
                }
            }
        }
    }
    }
    post {
        success { echo "Successfully deployed to ${params.REMOTE_HOST}" }
        failure { echo "Deployment failed. Check the console output for debugging." }
    }
}
```

### Checklist
- [x] Pipeline job created
- [x] Groovy script pasted
- [x] Parameters verified

---

## Phase 7: Run Pipeline

### Newton Server Prerequisites ✅

Verified all required tools on newton (192.168.8.53):

| Tool | Version | Status |
|------|---------|--------|
| Docker | 5.6.0 | ✅ |
| kubectl | v1.34.2 | ✅ |
| Helm | v3.19.2 | ✅ |
| dms_cli | /usr/local/bin/dms_cli | ✅ |

### Build History

**Build #1 — FAILURE**
```
Error: 'Jenkins' doesn't have label 'jenkins-agent-01'
Fix: Added label to built-in node in Manage Jenkins → Nodes
```

**Build #2 — FAILURE**
```
Error: Could not find credentials entry with ID 'gh_token'
Fix: Recreated GitHub credential as 'Username with password' type
```

**Build #3 — PARTIAL SUCCESS 🟡**

Pipeline ran successfully up to git clone:
```
✅ SSH connection to newton (192.168.8.53)
✅ Script transfer via SCP
✅ Remote script execution
✅ Environment setup on newton
🔴 git clone bmw-ece-ntust/kpimon-go-xapp — FAILED
```

Error:
```
remote: Repository not found.
fatal: repository 'https://github.com/bmw-ece-ntust/kpimon-go-xapp.git/' not found
```

>[!WARNING]
> 🔴 **Issue**: Repository `bmw-ece-ntust/kpimon-go-xapp` is private. Our GitHub token does not have access to the `bmw-ece-ntust` organization.
>
> **Status**: Access requested from lab admin (Ian). Pending approval.
>
> ⚠️ **Finding on Nino's doc**: The doc does not mention that the GitHub token needs organization membership or collaborator access to the private repo. This should be added as a prerequisite.

### Checklist
- [x] Newton prerequisites verified ✅
- [x] Pipeline runs and reaches newton ✅
- [x] SSH authentication works ✅
- [ ] git clone fails, pending repo access

---

## Summary of Findings on Nino's Documentation

| # | Finding | Severity | Recommendation |
|---|---------|----------|----------------|
| 1 | Doc uses `apt` commands but lab VMs run RHEL 9 which uses `dnf` | High | Update all install commands to use `dnf` |
| 2 | Jenkins 2.555.1 requires Java 21, but doc doesn't mention minimum Java version | High | Add Java version requirement explicitly |
| 3 | Doc assumes separate `jenkins-agent-01` node exists without explaining setup | Medium | Add step to configure agent label on built-in node |
| 4 | Credential Kind not specified — wrong type causes silent pipeline failure | High | Explicitly state Kind = "Username with password" for GitHub token |
| 5 | No mention that GitHub token needs org access to `bmw-ece-ntust` private repo | High | Add as prerequisite: request org access from lab admin |
| 6 | `REMOTE_PASS` parameter is deprecated per doc but still in pipeline script | Low | Remove or add note that it's unused |

---

## References

- [Jenkins Official Docs](https://www.jenkins.io/doc/)
- [Jenkins on RHEL Install Guide](https://www.jenkins.io/doc/book/installing/linux/#red-hat-centos)
- [Docker Install on RHEL](https://docs.docker.com/engine/install/rhel/)
- [Jenkins Java Support](https://jenkins.io/redirect/java-support/)
- [Nino's Jenkins CI/CD Handover Doc](jenkins-xapp-deployment-pipeline.md)
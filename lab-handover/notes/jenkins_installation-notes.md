# Jenkins CI/CD Installation Note
**VM 1: Jenkins Server**

> Author: Petrajoy Davidson | Date: 2026-04-26 | Status: ✅ Phase 0-2 Complete

---

## Job Checklist

| Phase | Status |
|-------|--------|
| Phase 0: VM Specs | ✅ Done |
| Phase 1: Install Prerequisites | ✅ Done |
| Phase 2: Install Jenkins | ✅ Done |
| Phase 3: Configure Jenkins Agent | ⏳ Pending |
| Phase 4: Install Required Plugins | ⏳ Pending |
| Phase 5:  Set Up Credentials | ⏳ Pending |

---

## Phase 0: VM Specifications

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

>[!NOTE]
> `ip a` showed 192.168.8.57 but the actual routing IP is 192.168.8.54. Always verify with `ip route show`.

- [x] Note down CPU cores, RAM, disk size, OS
- [x] Note down VM IP address

---

## Phase 1: Install Prerequisites


### Checklist

- [x] Get sudo access (admin ran: `usermod -aG wheel joy3`)
- [x] `dnf update -y`
- [x] `dnf install -y java-17-openjdk` *(later upgraded to Java 21, see troubleshooting)*
- [x] Verify Java: `java -version`
- [x] `dnf install -y git`
- [x] Verify Git: `git --version`
- [x] Install Docker:
  ```bash
  dnf install -y yum-utils
  yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
  dnf install -y docker-ce docker-ce-cli containerd.io
  ```
- [x] `systemctl enable --now docker`
- [x] Verify Docker: `docker ps`

### Versions Installed

| Package | Version |
|---------|---------|
| Java | OpenJDK 21.0.11 (upgraded from 17) |
| Git | 2.47.3 |
| Docker | 29.4.1 |
| containerd | 2.2.3 |

---

## Phase 2: Install Jenkins

>[!NOTE]
> Jenkins installed via official RPM repository for RHEL/Fedora-based systems.
> Jenkins 2.555.1 requires Java 21 minimum, Java 17 will not work.

### Installation Commands

```bash
# Add Jenkins repo
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

# Import GPG key
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Install Jenkins
dnf install -y jenkins

# Configure Java 21 for Jenkins (required! see troubleshooting)
mkdir -p /etc/systemd/system/jenkins.service.d/
cat > /etc/systemd/system/jenkins.service.d/override.conf << EOF
[Service]
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk-21.0.11.0.10-2.el9.x86_64"
EOF

# Reload and start Jenkins
systemctl daemon-reload
systemctl reset-failed jenkins
systemctl start jenkins

# Verify
systemctl status jenkins
curl http://localhost:8080
```

### Checklist

- [x] Add Jenkins RPM repository
- [x] Import Jenkins GPG key
- [x] `dnf install -y jenkins` → installed version 2.555.1
- [x] Configure Java 21 via systemd override
- [x] `systemctl start jenkins`
- [x] Verify Jenkins running: `systemctl status jenkins`
- [x] Verify via curl: `curl http://localhost:8080` → returns login page ✅
- [x] Access Jenkins UI at `http://192.168.8.54:8080` ✅
- [x] Unlock with `initialAdminPassword` ✅
- [x] Complete setup wizard ✅
- [x] Install suggested plugins ✅

### Jenkins Info

| Item | Value |
|------|-------|
| Version | 2.555.1 |
| URL | http://192.168.8.54:8080 |
| Initial Admin Password | `1328d516e4354d96a18569f8379512eb` |
| Password file location | `/var/lib/jenkins/secrets/initialAdminPassword` |

---

## Phase 3: Configure Jenkins Agent

- [ ] Go to Manage Jenkins → Nodes
- [ ] Create a node labeled exactly `jenkins-agent-01`
- [ ] Configure the agent workspace (`/root/jenkins/workspace/`)
- [ ] Verify agent is online and connected

---

## Phase 4: Install Required Jenkins Plugins

- [ ] Go to Manage Jenkins → Plugins
- [ ] Install: Pipeline
- [ ] Install: Git
- [ ] Install: Credentials Binding
- [ ] Install: SSH Agent
- [ ] Restart Jenkins after plugin installation

---

## Phase 5: Set Up Credentials

### Credential 1 — GitHub Token (`gh_token`)

- [ ] Generate GitHub Personal Access Token (repo read access)
- [ ] Go to Manage Jenkins → Credentials → Global
- [ ] Add credential: type = `Username with password`
- [ ] Set ID exactly as: `gh_token`
- [ ] Username = GitHub username, Password = token
- [ ] Save and verify

### Credential 2: SSH Private Key (`ssh-pubkey-kenny-192.168.8.53`)

- [ ] Generate SSH key pair: `ssh-keygen -t rsa -b 4096`
- [ ] Add public key to target server `~/.ssh/authorized_keys`
- [ ] Go to Manage Jenkins → Credentials → Global
- [ ] Add credential: type = `Secret file`
- [ ] Set ID exactly as: `ssh-pubkey-kenny-192.168.8.53`
- [ ] Upload private key file
- [ ] Save and verify

---

## Phase 6: Create Pipeline Job

- [ ] Click New Item → name it `Deploy_KPIMON_xApp` → choose Pipeline
- [ ] Paste Groovy script from Section 3.2 of Jenkins doc
- [ ] Verify default parameters:
  - `REMOTE_HOST`: `192.168.8.53`
  - `REMOTE_USER`: `kenny`
  - `GIT_CREDENTIAL_ID`: `gh_token`
  - `GIT_REPO_PATH`: `github.com/bmw-ece-ntust/kpimon-go-xapp.git`
  - `GIT_BRANCH`: `CICD`
  - `IMAGE_NAME`: `ric-app-kpimon-go`
  - `TAG`: `1.0.1`
  - `CHART_VERSION`: `2.0.1`
- [ ] Save the job

---

## Troubleshooting Notes

| # | Issue | Cause | Solution |
|---|-------|-------|----------|
| 1 | `joy3 is not in the sudoers file` | User not in wheel group | Admin ran: `usermod -aG wheel joy3` |
| 2 | `apt: command not found` | RHEL uses dnf not apt | Use `dnf` instead of `apt` |
| 3 | Jenkins service failed to start | Jenkins 2.555.1 requires Java 21, Java 17 was installed | `dnf install -y java-21-openjdk`, then `alternatives --config java` → select Java 21 |
| 4 | Jenkins still failed after Java 21 install | systemd service still pointing to Java 17 | Created `/etc/systemd/system/jenkins.service.d/override.conf` with correct `JAVA_HOME` |
| 5 | Jenkins stuck in failed state | Too many restart attempts | `systemctl reset-failed jenkins` then `systemctl start jenkins` |
| 6 | `curl http://192.168.8.57:8080` → No route to host | Wrong IP — `ip a` showed `.57` but routing IP is `.54` | Use `ip route show` to get correct IP → `192.168.8.54` ✅ |
| 7 | SSH tunnel `channel: open failed` | Tunnel issue | Not needed — direct browser access works on correct IP |

---

## References

- [Jenkins Official Docs](https://www.jenkins.io/doc/)
- [Jenkins on RHEL Install Guide](https://www.jenkins.io/doc/book/installing/linux/#red-hat-centos)
- [Docker Install on RHEL](https://docs.docker.com/engine/install/rhel/)
- [Jenkins Java Support](https://jenkins.io/redirect/java-support/)
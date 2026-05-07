# NFO Deployment Attempt Log
**Date**: 2026-05-07  
**Server**: `osc@192.168.8.76` (worker-rt)  
**Goal**: Duplicate Nino's NFO (Network Function Orchestrator) deployment following `smo-nfo-vnf-guide.md`

---

## Environment

The SMO is already installed and running on this server. We verified the cluster is healthy:

```bash
kubectl get pods -A
# Expected output: pods running in nonrtric, smo, onap, oai-cn, oai-ran namespaces
# Actual output: confirmed — all major components Running
```

---

## Attempt: Manual Installation (Option 1)

We went with manual installation because Docker/Podman was broken on this server (see note below).

>[!NOTE]
> Nino's guide provides a Docker Compose deployment option. However, `docker` was not installed, and `podman` (the available alternative) had a database configuration mismatch (`/var/lib/containers/storage/libpod` vs `/home/dockerdata/libpod`) that could not be resolved. Installing `podman-docker` and running `podman system reset --force` both failed with the same error. We proceeded with manual installation instead.

---

### Step 1: Clone Tacker

```bash
python3.11 -m venv /opt/tacker-venv-311
source /opt/tacker-venv-311/bin/activate
git clone https://opendev.org/openstack/tacker.git
cd tacker
```

> **Note — Python version**: Nino's guide specifies Python 3.8+. The server has Python 3.9, but Tacker's latest branch requires 3.10+. We installed Python 3.11 via `dnf install -y python3.11` to satisfy this.


```bash
pip install -e .
pip install python-keystoneclient python-openstackclient

# Expected output: Successfully installed tacker-x.x.x ...
```

We hit an issue with the latest Tacker branch and switched to `stable/2024.2`:

```bash
git checkout stable/2024.2
pip install -e .

# Expected output: Successfully installed tacker-12.0.1.dev2
```

---

### Step 2: Configure Tacker

Nino's guide references `tacker/etc/tacker/tacker.conf.sample` which does not exist in the cloned repo. We used the test config as a base:

```bash
mkdir -p /etc/tacker
cp /home/osc/tacker/samples/tests/etc/tacker.conf.test /etc/tacker/tacker.conf
```

We then updated it to match Nino's key configuration settings, with one change: `mysql` hostname replaced with `localhost` since there's no standalone MySQL on this server. We installed MariaDB locally:

```bash
dnf install -y mariadb-server
systemctl start mariadb
systemctl enable mariadb
mysql_secure_installation
```

Created the Tacker database and user:

```bash
mysql -u root <<EOF
CREATE DATABASE tacker;
CREATE USER 'tacker'@'localhost' IDENTIFIED BY 'tacker_password';
GRANT ALL PRIVILEGES ON tacker.* TO 'tacker'@'localhost';
FLUSH PRIVILEGES;
EOF
```

Final `/etc/tacker/tacker.conf`:

```ini
[DEFAULT]
debug = True
log_dir = /var/log/tacker
auth_strategy = keystone

[keystone_authtoken]
www_authenticate_uri = http://keystone:5000
auth_url = http://keystone:5000
project_domain_name = Default
user_domain_name = Default
project_name = admin
username = tacker
password = tacker_password
auth_type = password

[nfv]
nfv_drivers = kubernetes

[database]
connection = mysql+pymysql://tacker:tacker_password@localhost:3306/tacker
```

---

### Step 3: Initialize Database

```bash
mkdir -p /var/log/tacker
tacker-db-manage --config-file /etc/tacker/tacker.conf upgrade head

# Expected output: Running upgrade ... -> ca2ad037c320, add db tables for crypt key
# Actual output: All migrations ran successfully ✅
```

---

### Step 4: Start Tacker Services

Following Nino's guide:

```bash
tacker-server --config-file /etc/tacker/tacker.conf &
```

**Result: FAILED ❌**

```
TypeError: Element type must be a FieldType instance, not a class.
Use PackageOperationalStateType() instead of PackageOperationalStateType.
```

This is a compatibility bug between Tacker and `oslo.versionedobjects`. It occurs in `tacker/sol_refactored/objects/v2/pkgm_notification_filter.py` on both the latest branch and `stable/2024.2`.

`tacker-conductor` was not started as a result.

---

## Summary of Blockers

| # | Blocker | Impact |
|---|---------|--------|
| 1 | Podman database mismatch — Docker unavailable | Could not use Docker Compose deployment |
| 2 | Python 3.9 incompatible with latest Tacker | Required Python 3.11 workaround |
| 3 | `tacker.conf.sample` missing from repo | Used test config as base |
| 4 | Tacker crashes on startup — `oslo.versionedobjects` bug | Tacker server could not start |
| 5 | No Keystone on this server | Auth will fail even if Tacker starts |

---

## Next Steps

- Ask Nino for access to `bmw-ece-ntust/nino-o2-nfo` and clarify exactly how NFO was deployed on this server
- Investigate pinning `oslo.versionedobjects` to an older version to fix the startup crash
- Determine if Keystone is required or if auth can be disabled for testing
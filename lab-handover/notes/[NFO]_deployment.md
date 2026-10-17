# NFO Deployment Attempt Log
**Date**: 2026-05-07 to 2026-05-08 </br>
**Server**: `osc@192.168.8.76` (worker-rt)</br>
**Goal**: Duplicate Nino's NFO (Network Function Orchestrator) deployment following `smo-nfo-vnf-guide.md`

---

## Environment Check

First thing i did was verify the cluster is up. We SSHed in as `osc` but kubectl didn't work without root.

```bash
su -
kubectl get pods -A

# Output (partial):
# NAMESPACE   NAME                                                  READY   STATUS             RESTARTS   AGE
# nonrtric    rappmanager-0                                         1/1     Running            0          50d
# nonrtric    capifcore-6d557b5748-hnv6k                           1/1     Running            0          50d
# nonrtric    oran-nonrtric-kong-8d58799dd-kbmn5                   2/2     Running            6 (29d)    50d
# smo         keycloak-595475c5f4-hfxnz                            1/1     Running            0          50d
# smo         influxdb2-0                                          1/1     Running            0          50d
# onap        onap-policy-clamp-runtime-acm-5759d7fb6c-pn84p       1/1     Running            0          50d
# oai-cn      oai-amf-69989ddb4b-dvlmw                             2/2     Running            0          51d
# rapp        ta-rapp-nonrtric-rapp-test-automation-9ff947b79...   0/1     ErrImageNeverPull  0          20d
```

SMO is already installed and running. No NFO (Tacker) pods anywhere.

---

## Why Not Docker? (Tacker Installation)

Nino's guide provides a Docker Compose deployment option. Docker was not installed, so i tried:

```bash
dnf install -y docker

# Output:
# Installing: podman-docker-6:5.6.0-14.el9_7.noarch
# (installed podman-docker, a wrapper over Podman, not actual Docker)
```

Podman had a broken database configuration on this server:

```bash
docker ps -a

# Output:
# Error: database static dir "/var/lib/containers/storage/libpod" does not match
# our static dir "/home/dockerdata/libpod": database configuration mismatch
```

We tried `podman system reset --force` and removing `/var/lib/containers/storage/libpod` manually. Both failed with the same error. Docker/Podman is broken on this server and i moved on.

---

## Manual Installation (Option 1 from Nino's Guide)

### Step 1: Install Tacker

Nino's guide says Python 3.8+. The server has Python 3.9, but the latest Tacker requires 3.10+. I installed Python 3.11:

```bash
dnf install -y python3.11

# Output:
# Installed: python3.11-3.11.13-5.3.el9_7.x86_64
```

Created a venv and cloned upstream OpenStack Tacker :

```bash
python3.11 -m venv /opt/tacker-venv-311
source /opt/tacker-venv-311/bin/activate
git clone https://opendev.org/openstack/tacker.git
cd tacker
git checkout stable/2024.2
pip install -e .

# Output:
# Successfully installed tacker-12.0.1.dev2 oslo.privsep-3.11.0 oslo.rootwrap-7.9.0
# SQLAlchemy-2.0.49 oslo.versionedobjects-3.10.2 ...
```

Installed additional dependencies:

```bash
pip install python-keystoneclient python-openstackclient
pip install 'oslo.versionedobjects==3.8.0'

# Output:
# Successfully installed python-openstackclient-9.0.0 python-cinderclient-9.9.0
# Successfully installed oslo.versionedobjects-3.8.0
```

Note: the `oslo.versionedobjects` downgrade was needed to fix a crash on startup (see Step 4).

---

### Step 2: Configure Tacker

Nino's guide says to copy `tacker/etc/tacker/tacker.conf.sample`. That file does not exist:

```bash
cp /home/osc/tacker/etc/tacker/tacker.conf.sample /etc/tacker/tacker.conf

# Output:
# cp: cannot stat '/home/osc/tacker/etc/tacker/tacker.conf.sample': No such file or directory
```

I used the test config as a base instead:

```bash
mkdir -p /etc/tacker
cp /home/osc/tacker/samples/tests/etc/tacker.conf.test /etc/tacker/tacker.conf
```

No MySQL on the server, so i installed MariaDB locally:

```bash
dnf install -y mariadb-server
systemctl start mariadb
systemctl enable mariadb
mysql_secure_installation

mysql -u root <<EOF
CREATE DATABASE tacker;
CREATE USER 'tacker'@'localhost' IDENTIFIED BY 'tacker_password';
GRANT ALL PRIVILEGES ON tacker.* TO 'tacker'@'localhost';
FLUSH PRIVILEGES;
EOF

# Output:
# (no errors, commands ran successfully)
```

Also copied the missing `api-paste.ini` and added its path to `tacker.conf`:

```bash
cp /home/osc/tacker/etc/tacker/api-paste.ini /etc/tacker/
echo "api_paste_config = /etc/tacker/api-paste.ini" >> /etc/tacker/tacker.conf
```

Final `/etc/tacker/tacker.conf`:

```ini
[DEFAULT]
debug = True
log_dir = /var/log/tacker
auth_strategy = keystone
api_paste_config = /etc/tacker/api-paste.ini

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

# Output:
# INFO  [alembic.runtime.migration] Context impl MySQLImpl.
# INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
# INFO  [alembic.runtime.migration] Running upgrade  -> 1c6b0d82afcd, add tables for tacker framework
# INFO  [alembic.runtime.migration] Running upgrade 1c6b0d82afcd -> 81ffa86020d, rpc_proxy
# INFO  [alembic.runtime.migration] Running upgrade 81ffa86020d -> 4c31092895b8, empty message
# ...
# INFO  [alembic.runtime.migration] Running upgrade 4c0e2e2c2e02 -> ca2ad037c320, add db tables for crypt key
```

All migrations ran successfully.

---

### Step 4: Start Tacker Server

**Attempt 1** (before downgrading oslo.versionedobjects):

```bash
tacker-server

# Output:
# TypeError: Element type must be a FieldType instance, not a class.
# Use PackageOperationalStateType() instead of PackageOperationalStateType.
# (crash in tacker/sol_refactored/objects/v2/pkgm_notification_filter.py)
```

Fixed by downgrading `oslo.versionedobjects` to 3.8.0.

**Attempt 2** (after downgrade):

```bash
tacker-server --log-file /tmp/tacker.log

# Log output:
# ERROR oslo_service.backend._eventlet.service [-] Error starting thread.:
# oslo_config.cfg.ConfigFilesNotFoundError: Failed to find some config files: api-paste.ini
```

Fixed by copying `api-paste.ini` and adding `api_paste_config` to `tacker.conf`.

**Attempt 3** (final attempt):

```bash
tacker-server --log-file /tmp/tacker.log

# Log output:
# ERROR oslo_service.backend._eventlet.service [-] Error starting thread.:
# AttributeError: '_TransactionFactory' object has no attribute 'get_legacy_facade'
#
# Traceback:
#   File ".../oslo_service/backend/_eventlet/service.py", line 721, in run_service
#     service.start()
#   File ".../tacker/service.py", line 77, in start
#     self.wsgi_app = _run_wsgi(self.app_name)
#   File ".../tacker/service.py", line 117, in _run_wsgi
#     app = config.load_paste_app(app_name)
```

This is a SQLAlchemy 2.0 incompatibility. Tacker's `stable/2024.2` still calls `get_legacy_facade()` which was removed in SQLAlchemy 2.0. The pip install pulled SQLAlchemy 2.0.49 automatically. Stopped here.

---

## Summary of Blockers

| # | Blocker | Impact |
|---|---------|--------|
| 1 | Podman database mismatch, Docker unavailable | Could not use Docker Compose deployment |
| 2 | `tacker.conf.sample` missing from upstream repo | Used test config as base |
| 3 | `oslo.versionedobjects` incompatibility on latest branch | Required downgrade to 3.8.0 |
| 4 | `api-paste.ini` not found by Tacker at runtime | Required manual copy and config fix |
| 5 | SQLAlchemy 2.0 incompatibility (`get_legacy_facade` removed) | Tacker server crashes on startup |
| 6 | No Keystone on this server | Auth will fail even if Tacker starts |

---

## Conclusion

Nino's guide assumes a full OpenStack environment (Keystone, RabbitMQ, compatible library versions) that does not exist on this server. The guide also references a private repo we cannot access. I got as far as DB initialization and partial server startup, but Tacker could not fully start due to library version conflicts.

I need to ask Nino directly how NFO was actually deployed, and request access to the private repo.
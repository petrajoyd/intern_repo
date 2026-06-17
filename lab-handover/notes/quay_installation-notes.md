# Quay Container Registry Installation Note
**VM 2: Quay Server**

> Author: Joy | Date: 2026-04-27 | Status: ✅ Quay Running

---

## Job Checklist

| Phase | Status |
|-------|--------|
| Phase 0 — VM Specs | ✅ Done |
| Phase 1 — Install Prerequisites | ✅ Done |
| Phase 2 — Install Quay | ✅ Done |
| Phase 3 — Create Repository for OAI gNB | ⏳ Pending |
| Phase 4 — Connect Jenkins → Quay | ⏳ Pending |

---

## Phase 0: VM Specifications

| Spec | Value |
|------|-------|
| OS | Red Hat Enterprise Linux 9.4 (Plow) |
| CPU Cores | 8 cores |
| RAM | 7.5 GiB total / 5.9 GiB available |
| Swap | 5.0 GiB |
| Disk (root) | 44 GiB total / 39 GiB free |
| IP Address | 192.168.8.57 (ens192) |
| Hostname | localhost |
| Username | joy3 |

---

## Phase 1: Install Prerequisites


### Docker Installation

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
| Docker | 29.4.1 |
| containerd | 2.2.3 |

### Checklist

- [x] Add Docker repo
- [x] `dnf install -y docker-ce docker-ce-cli containerd.io`
- [x] `systemctl enable --now docker`
- [x] Verify: `docker ps`

---

## Phase 2: Install Quay

>[!NOTE]
> Quay installed via Docker container (simpler than Operator on K8s for lab use).
> Requires PostgreSQL and Redis as dependencies.

### Architecture

```
[Browser / Jenkins] → Quay (port 8080) → PostgreSQL (port 5432)
                                        → Redis (port 6379)
```

### Step 1 — Run Redis

```bash
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:latest

# Get Redis IP
docker inspect redis --format='{{.NetworkSettings.Networks.bridge.IPAddress}}'
# Result: 172.17.0.2
```

### Step 2 — Run PostgreSQL

```bash
docker run -d \
  --name postgres \
  -e POSTGRES_USER=quay \
  -e POSTGRES_PASSWORD=quaypass \
  -e POSTGRES_DB=quay \
  -p 5432:5432 \
  postgres:13

# Get PostgreSQL IP
docker inspect postgres --format='{{.NetworkSettings.Networks.bridge.IPAddress}}'
# Result: 172.17.0.3

# Install required pg_trgm extension
docker exec -it postgres psql -U quay -d quay -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"
```

### Step 3 — Create Quay Config

```bash
mkdir -p /opt/quay/config

cat > /opt/quay/config/config.yaml << EOF
AUTHENTICATION_TYPE: Database
BUILDLOGS_REDIS:
    host: 172.17.0.2
    port: 6379
USER_EVENTS_REDIS:
    host: 172.17.0.2
    port: 6379
DATABASE_SECRET_KEY: asecretkey
DB_URI: postgresql://quay:quaypass@172.17.0.3/quay
DISTRIBUTED_STORAGE_CONFIG:
    default:
        - LocalStorage
        - storage_path: /datastorage/registry
DISTRIBUTED_STORAGE_DEFAULT_LOCATIONS: []
DISTRIBUTED_STORAGE_PREFERENCE:
    - default
SECRET_KEY: asecretkey
SERVER_HOSTNAME: 192.168.8.57:8080
SETUP_COMPLETE: true
FEATURE_MAILING: false
FEATURE_REQUIRE_ENCRYPTED_BASIC_AUTH: false
EOF
```

### Step 4 — Run Quay

```bash
docker run -d \
  --name quay \
  -p 8080:8080 \
  -p 443:8443 \
  -v /opt/quay/config:/conf/stack \
  quay.io/projectquay/quay:latest

sleep 15
curl http://localhost:8080
```

### Checklist

- [x] Run Redis container
- [x] Run PostgreSQL container
- [x] Install pg_trgm extension in PostgreSQL
- [x] Create Quay config file
- [x] Run Quay container
- [x] Verify Quay accessible at `http://192.168.8.57:8080` ✅
- [x] Create admin account (username: `petrajoyd`) ✅

### Quay Info

| Item | Value |
|------|-------|
| Version | v3.8.1 |
| URL | http://192.168.8.57:8080 |
| Admin User | petrajoyd |
| PostgreSQL | 172.17.0.3:5432 |
| Redis | 172.17.0.2:6379 |

---

## Phase 3: Create Repository for OAI gNB

- [x] Go to `http://192.168.8.57:8080`
- [x] Click **+ Create New Repository**
- [x] Name: `ric-app-kpimon-go`
- [x] Visibility: Public
- [x] Created successfully ✅

---

## Phase 4: Connect Jenkins → Quay

### Configure Docker Insecure Registry on Jenkins VM
 
```bash
cat > /etc/docker/daemon.json << EOF
{
  "insecure-registries": ["192.168.8.57:8080"]
}
EOF
 
systemctl restart docker
```
 
### Login to Quay from Jenkins VM
 
```bash
docker login 192.168.8.57:8080 -u petrajoyd
# Login Succeeded ✅
```
 
### Test Push from Jenkins VM to Quay
 
```bash
docker pull hello-world
docker tag hello-world 192.168.8.57:8080/petrajoyd/ric-app-kpimon-go:test
docker push 192.168.8.57:8080/petrajoyd/ric-app-kpimon-go:test
# Push succeeded ✅
```
 
### Checklist
 
- [x] Configure insecure registry on Jenkins VM
- [x] `docker login 192.168.8.57:8080 -u petrajoyd` ✅
- [x] Test push image to Quay ✅
- [x] Verify image appears in Quay UI ✅

---

## ✅ Summary — Jenkins → Quay Pipeline Working!
 
| Component | Status | URL |
|-----------|--------|-----|
| Jenkins | ✅ Running | http://192.168.8.54:8080 |
| Quay | ✅ Running | http://192.168.8.57:8080 |
| Jenkins → Quay push | ✅ Working | - |
 
>[!NOTE] 
>  Quay containers (quay, redis, postgres) do NOT auto-start on VM reboot.
> Run these manually after reboot:
> ```bash
> docker start redis postgres quay
> ```

---

## Troubleshooting Notes

| # | Issue | Cause | Solution |
|---|-------|-------|----------|
| 1 | Quay Operator not found in catalog | `stable-3.13` channel doesn't exist in community catalog | Switched to Docker-based installation |
| 2 | Quay crashed: `no such file or directory` | Config file missing | Created `/opt/quay/config/config.yaml` |
| 3 | Quay crashed: invalid DB scheme | SQLite not supported by Quay | Switched to PostgreSQL |
| 4 | Quay crashed: `pg_trgm` missing | Required PostgreSQL extension not installed | `CREATE EXTENSION IF NOT EXISTS pg_trgm` |
| 5 | Redis 🔴 in config check | `USER_EVENTS_REDIS` missing from config | Added `USER_EVENTS_REDIS` to config.yaml |
| 6 | Email verification stuck | No mail server configured | Set `FEATURE_MAILING: false` in config |
| 7 | Account stuck as unverified | Email verification required but no email server | Manually verified in DB: `UPDATE public.user SET verified=true WHERE username='petrajoyd'` |

---

## References

- [Project Quay Docs](https://docs.projectquay.io/)
- [Quay GitHub](https://github.com/quay/quay)
- [PostgreSQL Docker Hub](https://hub.docker.com/_/postgres)
- [Redis Docker Hub](https://hub.docker.com/_/redis)
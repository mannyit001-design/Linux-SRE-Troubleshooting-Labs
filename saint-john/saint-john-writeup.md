# 🔎 Saint John — What Is Writing to This Log File?

**Platform:** SadServers | **Level:** Easy | **Time to Solve:** 10 minutes | **OS:** Debian 11

---

## 📋 Scenario

> A developer created a testing program that is continuously writing to a log file `/var/log/bad.log` and filling up disk. The program is no longer needed. Find it and terminate it. Do not delete the log file.

**Success condition:** the log file's size stops changing (verified over a time interval longer than its rate of growth).

---

## 🩺 Diagnosis

The problem statement gives one hint immediately: something is *actively writing* to a specific file. On Linux, the right tool to answer "what process currently has this file open" is `lsof` (list open files).

```bash
sudo lsof /var/log/bad.log
```

Output:
```
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF   NODE NAME
badlog.py 596 admin    3w   REG  259,1    47503 265802 /var/log/bad.log
```

This immediately identifies everything needed:
- **COMMAND**: `badlog.py` — a Python script
- **PID**: `596`
- **FD (file descriptor) 3w**: file descriptor 3, opened in **w**rite mode — confirming this process is the one actively writing

---

## 🔧 Fix

With the PID identified, the next decision is *how* to stop it. `kill` sends a signal to a process:
- Plain `kill` (no flag) sends **SIGTERM** — a polite request to shut down, allowing the process to clean up
- `kill -9` sends **SIGKILL** — an immediate, forceful termination with no chance to clean up

Since there was no indication the process needed a hard kill, the safer first move is a plain `kill`:

```bash
sudo kill 596
```

---

## ✅ Verification

Two checks confirm the fix:

```bash
sudo lsof /var/log/bad.log
```
No output — no process has the file open anymore.

```bash
tail -f /var/log/bad.log
```
The file stops growing (Ctrl+C to exit the follow).

The log file itself was left in place, per the requirement — only the process writing to it was terminated.

---

## 🧠 Key Takeaway

`lsof` is the fastest way to answer "what's touching this file/port right now" — a core skill for any live troubleshooting where a resource is being held or modified by an unknown process. Preferring `kill` over `kill -9` as a default first step is also a habit worth keeping: graceful termination avoids leaving resources (locks, partial writes, temp files) in a bad state.

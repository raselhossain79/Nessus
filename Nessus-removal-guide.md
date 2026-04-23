# 🧹 Nessus Complete Removal Guide — Kali Linux

## 📌 Overview

This guide walks through the complete removal of **Tenable Nessus** from a Kali Linux system — including stopping the service, purging the package, deleting leftover files, and verifying that port `8834` is fully closed.

> **Target OS:** Kali Linux (Debian-based)
> **Nessus Default Port:** `8834`
> **Requires:** `sudo` / root access

---

## ⚠️ Before You Start

- Close the Nessus Web UI in your browser (`https://kali:8834`)
- Make sure you have terminal access with `sudo` privileges
- No active scans should be running

---

## Step 1 — Stop the Nessus Service

```bash
sudo systemctl stop nessusd
```

**What this does:**
Sends a stop signal to the `nessusd` daemon (Nessus background service). This gracefully shuts down the service so it stops listening on port `8834`. Without this step, the process stays alive in memory even if you try to remove the package.

---

## Step 2 — Disable the Service (Prevent Auto-Start)

```bash
sudo systemctl disable nessusd
```

**What this does:**
Removes the symlink that tells `systemd` to start `nessusd` on boot. Even if the package is still installed, the service will no longer start automatically on system reboot.

> **Verify:** `sudo systemctl status nessusd` → should show `inactive (dead)`

---

## Step 3 — Purge the Nessus Package

```bash
sudo apt purge nessus -y
```

**What this does:**
`purge` goes beyond a normal `remove` — it uninstalls the package **and** deletes any configuration files managed by the package manager. The `-y` flag auto-confirms the prompt.

> **Difference between `remove` vs `purge`:**
> - `apt remove` → removes binaries only
> - `apt purge` → removes binaries **+** config files tracked by apt

---

## Step 4 — Delete Leftover Files Manually

```bash
sudo rm -rf /opt/nessus
sudo rm -rf /etc/nessus
sudo rm -rf /var/nessus
```

**What this does:**

| Path | Contents |
|------|----------|
| `/opt/nessus` | Main Nessus installation directory (plugins, binaries, web UI files) |
| `/etc/nessus` | System-level configuration files (not always cleaned by purge) |
| `/var/nessus` | Runtime data — logs, scan data, user accounts, license info |

> ⚠️ `rm -rf` is irreversible. Double-check paths before running.

---

## Step 5 — Clean Orphaned Dependencies

```bash
sudo apt autoremove -y
```

**What this does:**
Removes any packages that were installed as dependencies for Nessus but are no longer needed by any other package on the system. Keeps your system clean.

---

## Step 6 — Verify Port 8834 is Closed

```bash
sudo ss -tulnp | grep 8834
```

**What this does:**
`ss` is the modern replacement for `netstat`. This command lists all open TCP/UDP ports and filters for port `8834`.

**Reading the output:**

| Result | Meaning |
|--------|---------|
| No output | ✅ Port is closed — Nessus fully removed |
| Shows a process | ⚠️ A process is still running on port 8834 — go to Step 7 |

---

## Step 7 — Force Kill Remaining Process (If Needed)

If Step 6 shows output, find the PID and kill it:

```bash
# First, get the PID
sudo ss -tulnp | grep 8834

# Example output:
# tcp LISTEN 0 128 0.0.0.0:8834 ... users:(("nessusd",pid=1234,fd=6))
#                                                          ^^^^ PID is here

# Kill the process
sudo kill -9 <PID>
```

**What this does:**
`kill -9` sends `SIGKILL` — an immediate, forceful termination signal that cannot be ignored or caught by the process. The process is killed instantly by the kernel.

> Replace `<PID>` with the actual process ID from the output above (e.g., `sudo kill -9 1234`)

---

## Step 8 — Browser Verification

Open your browser and go to:

```
https://kali:8834
```

**Expected results:**

| Result | Action |
|--------|--------|
| "Site can't be reached" / connection refused | ✅ Nessus fully removed |
| Page still loads | Try incognito mode or clear browser cache (could be cached) |
| Page still loads in incognito | Recheck Step 6 — a process may still be running |

---

## ✅ Final Checklist

| Task | Status |
|------|--------|
| `nessusd` service stopped | ✅ |
| Auto-start disabled | ✅ |
| Package purged via apt | ✅ |
| `/opt/nessus` deleted | ✅ |
| `/etc/nessus` deleted | ✅ |
| `/var/nessus` deleted | ✅ |
| Port `8834` confirmed closed | ✅ |

---

## ⚡ Quick Reference — All Commands at Once

Run these in order for a clean, complete removal:

```bash
# 1. Stop and disable service
sudo systemctl stop nessusd
sudo systemctl disable nessusd

# 2. Purge package
sudo apt purge nessus -y

# 3. Remove leftover files
sudo rm -rf /opt/nessus /etc/nessus /var/nessus

# 4. Clean orphaned packages
sudo apt autoremove -y

# 5. Verify port 8834 is closed
sudo ss -tulnp | grep 8834
# No output = fully removed ✅
```

> If port `8834` still shows a process after all steps, kill it:
> ```bash
> sudo kill -9 <PID>
> ```

---

*Guide written for Kali Linux. Commands tested on Debian-based systems with systemd.*

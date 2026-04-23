# 🛡️ Nessus Complete Installation Guide — Kali Linux

## 📌 Overview

This guide covers the complete installation and setup of **Tenable Nessus Essentials** on Kali Linux — from downloading the package to completing the web-based setup wizard and running your first scan.

> **Target OS:** Kali Linux (Debian-based)
> **Version Used:** Nessus Essentials (Free — up to 16 IPs)
> **Nessus Default Port:** `8834`
> **Requires:** `sudo` / root access + internet connection

---

## 📋 Prerequisites

| Requirement | Details |
|-------------|---------|
| OS | Kali Linux (64-bit recommended) |
| RAM | Minimum 4 GB (8 GB recommended) |
| Disk Space | ~3 GB free minimum |
| Internet | Required for download + activation |
| Account | Free Tenable account (for activation code) |

---

## Step 1 — Get a Free Activation Code

Before downloading, you need a **free activation code** from Tenable.

1. Go to: [https://www.tenable.com/products/nessus/nessus-essentials](https://www.tenable.com/products/nessus/nessus-essentials)
2. Fill in the registration form (name + email)
3. Tenable will email you an **activation code** — save it, you'll need it later

> **Nessus Essentials** is free for students and learning. Allows scanning up to **16 IP addresses**.

---

## Step 2 — Download Nessus Package

Go to the official Tenable download page:

🔗 [https://www.tenable.com/downloads/nessus](https://www.tenable.com/downloads/nessus)

**Select the correct package:**

| Field | Value |
|-------|-------|
| Product | Nessus |
| Version | Latest stable |
| Platform | Linux — Debian — amd64 |
| File | `Nessus-<version>-debian10_amd64.deb` |

**Or download directly via terminal (get the link from the download page):**

```bash
# Navigate to Downloads folder
cd ~/Downloads

# Download using wget (replace URL with the actual link from Tenable's site)
wget -O nessus.deb "https://www.tenable.com/downloads/api/v1/public/pages/nessus/downloads/<ID>/download?i_agree_to_tenable_license_agreement=true"
```

> ⚠️ The direct download URL changes with each version. Always get the latest link from the official Tenable download page.

---

## Step 3 — Install the Package

```bash
sudo dpkg -i nessus.deb
```

**What this does:**
`dpkg -i` installs a `.deb` package directly (bypassing apt). It extracts all Nessus files into their correct locations:

| Path | What Gets Installed |
|------|---------------------|
| `/opt/nessus/` | Main installation (binaries, plugins, web UI) |
| `/etc/init.d/nessusd` | Init script |
| `/lib/systemd/system/nessusd.service` | Systemd service unit file |

**Expected output at the end:**
```
Unpacking nessus...
Setting up nessus...
nessusd (Nessus) 10.x.x. for Linux
Copyright (C) 1998-20xx Tenable Network Security...
```

---

## Step 4 — Start the Nessus Service

```bash
sudo systemctl start nessusd
```

**What this does:**
Starts the `nessusd` daemon. The service:
- Launches the Nessus backend engine
- Starts listening on port `8834` (HTTPS)
- Prepares the web UI for access

**Verify the service is running:**

```bash
sudo systemctl status nessusd
```

**Expected output:**
```
● nessusd.service - The Nessus Vulnerability Scanner
   Active: active (running) since ...
```

---

## Step 5 — Enable Auto-Start on Boot

```bash
sudo systemctl enable nessusd
```

**What this does:**
Creates a symlink so `systemd` automatically starts `nessusd` every time the system boots. Without this, you'd have to manually start Nessus after every reboot.

---

## Step 6 — Verify Port 8834 is Listening

```bash
sudo ss -tulnp | grep 8834
```

**Expected output:**
```
tcp   LISTEN  0   128   0.0.0.0:8834   0.0.0.0:*   users:(("nessusd",pid=XXXX,fd=6))
```

| Result | Meaning |
|--------|---------|
| Shows `LISTEN` on port `8834` | ✅ Nessus is running and ready |
| No output | ⚠️ Service didn't start — re-run Step 4 |

---

## Step 7 — Access the Web UI

Open your browser and navigate to:

```
https://kali:8834
```

or

```
https://localhost:8834
```

> ⚠️ **SSL Warning is normal** — Nessus uses a self-signed certificate. Click **"Advanced" → "Accept the Risk and Continue"** (Firefox) or **"Proceed anyway"** (Chrome).

---

## Step 8 — Complete the Setup Wizard

Once the web UI loads, follow the setup wizard:

### 8.1 — Choose Product Type

Select **"Nessus Essentials"** (free option).

---

### 8.2 — Enter Activation Code

Enter the activation code you received via email in Step 1.

```
XXXX-XXXX-XXXX-XXXX-XXXX
```

Click **"Continue"**.

---

### 8.3 — Create Admin Account

Create your Nessus admin credentials:

| Field | Recommendation |
|-------|----------------|
| Username | `admin` or your preferred name |
| Password | Strong password — save it somewhere safe |

> ⚠️ If you forget this password, you'll need to reset Nessus entirely.

---

### 8.4 — Plugin Compilation (Wait Here)

After account creation, Nessus will:
1. Download all vulnerability plugins from Tenable's servers
2. Compile and index them

**This takes 10–30 minutes depending on your internet speed.**

You'll see a progress bar — just leave it running. Do not close the browser or stop the service.

---

## Step 9 — First Login & Verify

Once plugin compilation is done:

1. Log in with the admin credentials you created
2. You'll land on the **Nessus Dashboard**
3. Go to **Settings → About** to confirm version and license

**You're in. ✅**

---

## 🔍 Useful Commands After Installation

```bash
# Check service status
sudo systemctl status nessusd

# Restart service (if needed)
sudo systemctl restart nessusd

# Stop service
sudo systemctl stop nessusd

# Check what's running on port 8834
sudo ss -tulnp | grep 8834

# View Nessus logs (useful for troubleshooting)
sudo tail -f /opt/nessus/var/nessus/logs/nessusd.messages
```

---

## ⚠️ Common Problem: "Nessus has no plugins. Therefore functionality is limited."

This warning appears in **Settings** after installation. It means Nessus could not download its vulnerability plugin database from Tenable's servers.

**Why it happens:**
- Lab network is blocking outbound connections to `plugins.nessus.org`
- Firewall or proxy restrictions on the machine
- No internet access on that specific PC

There are two ways to fix this:

---

### Fix Option 1 — Force Update via Command Line

When the GUI **Settings → Update** does nothing, try forcing the update directly from terminal.

```bash
# Step 1: Stop the service first
sudo systemctl stop nessusd

# Step 2: Force plugin update via CLI
sudo /opt/nessus/sbin/nessuscli update --all

# Step 3: Start the service again
sudo systemctl start nessusd
```

**What this does:**
`nessuscli update --all` bypasses the web UI and directly contacts Tenable's update servers from the command line. It downloads:
- All vulnerability plugins
- Plugin feed metadata
- Scanner engine updates (if available)

**After running, wait 5–10 minutes**, then refresh `https://localhost:8834` and check Settings.

**Verify activation is valid before trying:**
```bash
sudo /opt/nessus/sbin/nessuscli fetch --check
```

Expected output:
```
Fetching the newest updates from nessus.org...
Activation Code: Valid
```

If activation shows `Invalid` or `Not registered` → re-register via the web UI setup wizard first, then try the update again.

---

### Fix Option 2 — Offline Plugin Update (Best for Lab/No Internet)

When the PC has **no internet access at all**, download the plugin package from a different machine and transfer it manually. This is the most reliable fix in lab environments.

**Step 1 — Download plugin package from an internet-connected machine**

Go to the official Tenable download page:
```
https://www.tenable.com/downloads/nessus
```

Scroll down to find **"Nessus Offline Update"** section. Download the file:
```
all-2.0.tar.gz   (or similar name — this is the full plugin bundle)
```

> File size is typically **200–500 MB** depending on the version.

**Step 2 — Transfer the file to the target machine**

Use a USB drive, `scp`, shared folder (VMware shared folders), or any file transfer method available in your lab.

```bash
# If transferring via scp from another machine on the same network:
scp all-2.0.tar.gz kali@<target-ip>:~/Downloads/
```

**Step 3 — Apply the offline plugin update**

```bash
# Stop the service
sudo systemctl stop nessusd

# Apply the plugin package
sudo /opt/nessus/sbin/nessuscli update ~/Downloads/all-2.0.tar.gz

# Start the service
sudo systemctl start nessusd
```

**What this does:**
`nessuscli update <file>` extracts and installs the plugin bundle directly from the local `.tar.gz` file — no internet needed. Nessus will then compile and index all plugins internally.

**Step 4 — Wait for compilation**

After the command completes, Nessus still needs to compile the plugins in the background. This can take **10–20 minutes**.

Monitor progress from terminal:
```bash
sudo tail -f /opt/nessus/var/nessus/logs/nessusd.messages
```

Look for lines mentioning `plugin compilation` or `plugins loaded` — that means it's working.

**Step 5 — Verify in web UI**

Go to `https://localhost:8834` → **Settings → Overview**

You should now see:
```
Plugin Set: XXXXXXXXXX (a timestamp-based number)
```
Instead of the "no plugins" warning. ✅

---

> 💡 **Which option to use?**
> - Machine has internet but GUI update fails → **Try Option 1 first**
> - Machine has no internet / lab network is restricted → **Use Option 2**

---

## 🐛 Common Issues & Fixes

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| Web UI not loading | Service not started | `sudo systemctl start nessusd` |
| "Connection refused" error | Wrong port or service down | Verify port with `ss -tulnp \| grep 8834` |
| SSL error in browser | Self-signed cert | Click "Advanced" → accept the risk |
| **"Nessus has no plugins"** | **Lab network blocking `plugins.nessus.org`** | **See Plugin Fix section above** |
| Plugin download stuck | Slow internet / timeout | Wait longer or restart service |
| Forgot admin password | — | `sudo /opt/nessus/sbin/nessuscli lsuser` then reset |
| `dpkg` dependency errors | Missing libs | Run `sudo apt --fix-broken install` then retry |

---

## ✅ Final Checklist

| Task | Status |
|------|--------|
| Activation code received from Tenable | ✅ |
| `.deb` package downloaded | ✅ |
| Package installed with `dpkg` | ✅ |
| `nessusd` service started | ✅ |
| Auto-start on boot enabled | ✅ |
| Port `8834` confirmed listening | ✅ |
| Web UI accessible in browser | ✅ |
| Admin account created | ✅ |
| Plugins downloaded & compiled | ✅ |

---

## ⚡ Quick Reference — All Commands at Once

```bash
# 1. Install the downloaded package
sudo dpkg -i nessus.deb

# 2. Start the service
sudo systemctl start nessusd

# 3. Enable auto-start on boot
sudo systemctl enable nessusd

# 4. Verify service is running
sudo systemctl status nessusd

# 5. Verify port 8834 is listening
sudo ss -tulnp | grep 8834
# Should show LISTEN on port 8834 ✅

# 6. Open browser and go to:
# https://localhost:8834
```

> After that, complete the web UI setup wizard:
> Nessus Essentials → Enter activation code → Create admin account → Wait for plugin compilation

---

*Guide written for Kali Linux. Nessus is a product of Tenable, Inc. Always download from the official Tenable website.*

# Nessus🧹 Nessus Complete Removal Guide (Kali Linux)

📌 Overview

এই গাইডে দেখানো হয়েছে কিভাবে Kali Linux থেকে Nessus পুরোপুরি remove করতে হয়, যাতে service, files এবং port সব clean হয়ে যায়।

---

⚠️ Before Start

- Nessus web UI ("https://kali:8834") বন্ধ করে নাও
- Terminal access থাকতে হবে (root/sudo)

---

🧨 Step 1: Stop Service

sudo systemctl stop nessusd

---

🚫 Step 2: Disable Service

sudo systemctl disable nessusd

---

🗑️ Step 3: Remove Package

sudo apt purge nessus -y

---

🧼 Step 4: Remove Leftover Files

sudo rm -rf /opt/nessus
sudo rm -rf /etc/nessus
sudo rm -rf /var/nessus

---

🧹 Step 5: Clean System

sudo apt autoremove -y

---

🔍 Step 6: Check Port 8834 (Important)

sudo ss -tulnp | grep 8834

👉 যদি কিছু না আসে → fully removed ✅
👉 যদি process থাকে → kill করতে হবে

---

💀 Step 7: Kill Running Process (if needed)

sudo kill -9 <PID>

(PID বের হবে আগের command থেকে)

---

🌐 Step 8: Browser Check

- "https://kali:8834" খুললে যদি কিছু না আসে → success ✅
- যদি page আসে → cache clear / incognito try করো

---

💡 Final Result

✔ Service stopped
✔ Package removed
✔ Files deleted
✔ Port closed

👉 Nessus fully removed from system 🧹

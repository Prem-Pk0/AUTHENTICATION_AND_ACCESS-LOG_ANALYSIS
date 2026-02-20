# 🛡️ SMB Credential Stuffing Detection Lab (Splunk SOC Project)

---

# 📌 Project Overview

This lab simulates a real-world **SMB credential stuffing attack** and demonstrates how a SOC analyst can detect it using **Splunk SIEM**.

The project includes:

- Windows victim machine (VM2)
- Kali Linux attacker machine (VM1)
- Splunk SIEM server (VM3)
- Windows Security log ingestion
- Detection engineering
- Alert creation
- Dashboard visualization
- MITRE ATT&CK mapping

---

# 🌐 What is SMB?

**SMB (Server Message Block)** is a network protocol used for:

- File sharing
- Printer sharing
- Remote administration
- Windows authentication

SMB typically runs on:

```
TCP Port 445
```

When attackers target SMB, they often attempt:

- Password spraying
- Credential stuffing
- Brute-force attacks

---

# 🔐 What is Credential Stuffing?

Credential stuffing is when an attacker:

- Uses **one known password**
- Tries it against **multiple usernames**
- From the same source IP
- Within a short time window

If one account succeeds → compromise occurs.

MITRE ATT&CK Technique:

```
T1110.004 — Credential Stuffing
```

---

# 🖥️ PHASE 1 — Victim Setup (Windows VM2)

This phase prepares the Windows machine to:

- Log authentication attempts
- Send logs to Splunk
- Create test accounts for attack simulation

---

## 🔹 Enable Required Audit Policy

This ensures Windows logs both successful and failed login attempts.

Run as Administrator:

```
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Account Lockout" /success:enable /failure:enable
auditpol /get /subcategory:"Logon"
```

---

## 👤 Create Local Test Users

These accounts will be targeted during the attack simulation.

```
net user alice Password1! /add
net user bob Password2! /add
net user john Welcome@123 /add
net user test Password3! /add
```

---

## 📁 Navigate to Splunk Universal Forwarder

```
cd "C:\Program Files\SplunkUniversalForwarder\bin"
```

---

## ⚙️ Configure Splunk Universal Forwarder

This allows Windows Security logs to be sent to the Splunk Indexer.

### Edit inputs.conf

```
notepad "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf"
```

Add:

```
[WinEventLog://Security]
index=windows
disabled=0
```

---

### Edit outputs.conf

```
notepad "C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf"
```

Add:

```
[tcpout]
defaultGroup=splunk_indexer

[tcpout:splunk_indexer]
server=192.168.56.30:9997
```

---

## 🔄 Restart Splunk Forwarder

```
.\splunk.exe restart
```

---

## 📡 Verify Forwarding Status

```
.\splunk.exe list forward-server
```

---

## 🛡️ Add John to Administrators Group

This simulates a high-value account compromise scenario.

```
net localgroup administrators john /add
net localgroup administrators
```

---

# 🔍 PHASE 2 — Attack Execution (Kali Linux VM1)

In this phase, we simulate credential stuffing using CrackMapExec.

---

## 🔎 Verify SMB Port 445

```
nmap -p 445 192.168.56.20
```

---

## 📦 Install CrackMapExec

```
sudo apt update
sudo apt install crackmapexec -y
```

---

## 📝 Create Username Wordlist

```
nano users.txt
```

Add:

```
alice
bob
john
test
```

---

## 🔥 Launch Credential Stuffing Attack

```
crackmapexec smb 192.168.56.20 -u users.txt -p "Welcome@123"
```

Additional enumeration commands:

```
crackmapexec smb 192.168.56.20 -u john -p Welcome@123 --shares --local-auth
crackmapexec smb 192.168.56.20 -u john -p Welcome@123 --admin --local-auth
crackmapexec smb 192.168.56.20 -u john -p Welcome@123 --sessions --local-auth
crackmapexec smb 192.168.56.20 -u john -p Welcome@123 --users --local-auth
crackmapexec smb 192.168.56.20 -u john -p Welcome@123 --groups --local-auth
```

---

# 📊 PHASE 3 — Understanding Windows Event IDs

Important Windows Security Event Codes:

```
4625 — Failed Logon
4624 — Successful Logon
Logon_Type 3 — Network Logon (SMB)
```

These events are critical for detecting credential attacks.

---

# 🔍 PHASE 4 — Base Detection (Failed SMB Logins)

Run in Splunk:

```
index=wineventlog EventCode=4625 Logon_Type=3
| stats count by Account_Name, Source_Network_Address
```

This shows:

- Who failed authentication
- From which IP
- Over SMB

---

# 🔥 PHASE 5 — Credential Stuffing Detection Logic

## Detection Strategy

We look for:

- Same source IP
- Multiple different usernames
- All failed logins
- Within short time period

---

## ✅ SOC-Ready SPL Query

```
index=wineventlog EventCode=4625 Logon_Type=3
| where Source_Network_Address!="-"
| stats count dc(Account_Name) as unique_users values(Account_Name) as users by Source_Network_Address
| where unique_users >= 3
```

Explanation:

- dc(Account_Name) → distinct usernames
- unique_users >= 3 → threshold
- Same IP targeting multiple users

This strongly indicates credential stuffing.

---

# ✅ PHASE 6 — Confirm Account Compromise

We check if failures were followed by success.

```
index=wineventlog (EventCode=4625 OR EventCode=4624) Logon_Type=3
| stats count values(EventCode) by Account_Name, Source_Network_Address
```

SOC Investigation Pattern:

- Multiple 4625
- Followed by 4624
- Same Source IP

➡️ Confirmed compromise

---

# 🚨 PHASE 7 — Create Splunk Alert

## Alert Name

SMB Credential Stuffing Detected

## Detection Query

```
index=wineventlog EventCode=4625 Logon_Type=3
| stats dc(Account_Name) as unique_users by Source_Network_Address
| where unique_users >= 3
```

### Alert Settings

- Trigger: Once per result
- Time Window: 5 minutes
- Severity: High
- Throttle: 30 minutes

---

# 📈 PHASE 8 — SOC Dashboard Panels

## Panel 1 — SMB Failures Over Time

```
index=wineventlog EventCode=4625 Logon_Type=3
| timechart count
```

---

## Panel 2 — Top Source IPs

```
index=wineventlog EventCode=4625 Logon_Type=3
| stats count by Source_Network_Address
| sort -count
```

---

## Panel 3 — Targeted Users

```
index=wineventlog EventCode=4625 Logon_Type=3
| stats count by Account_Name
```

---

# 🧠 MITRE ATT&CK Mapping

```
T1110.004 — Credential Stuffing
T1078 — Valid Accounts
```

Framework: MITRE ATT&CK

---

# 🎯 Final Outcome

✔ Simulated real-world SMB credential stuffing  
✔ Collected Windows Security logs into Splunk  
✔ Built detection engineering logic  
✔ Created SOC alert  
✔ Designed monitoring dashboard  
✔ Mapped attack to MITRE ATT&CK  

---

# 🏆 Skills Demonstrated

- Windows Log Analysis
- Splunk SPL Development
- SIEM Alert Engineering
- Attack Simulation
- MITRE ATT&CK Mapping
- SOC Investigation Workflow

---

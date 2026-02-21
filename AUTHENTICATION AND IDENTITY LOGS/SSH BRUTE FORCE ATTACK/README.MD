<p align="center">
  <img src="https://res.cloudinary.com/dfw87bbyp/image/upload/v1771656054/WAZUH_xnmvbk.jpg" />
</p>

<h1 align="center">🔐 SSH Brute Force Detection Lab (Wazuh SOC Practice)</h1>

<p align="center">
  <img src="https://img.shields.io/badge/SIEM-Wazuh-blue?style=for-the-badge&logo=gnu" />
  <img src="https://img.shields.io/badge/Attack-SSH%20Brute%20Force-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Platform-Linux-orange?style=for-the-badge&logo=linux" />
  <img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-Apache%202.0-lightgrey?style=for-the-badge" />
</p>

---

# 📌 Lab Overview

This lab simulates a real-world **SSH brute force attack** and demonstrates how a SOC analyst can detect and investigate it using **Wazuh SIEM**.

The Lab includes:

- Linux victim machine (VM2)
- Kali Linux attacker machine (VM1)
- Wazuh Manager + Agent (VM3)
- SSH service configuration
- Weak credential simulation
- Brute force attack execution (Hydra)
- Log analysis from `/var/log/auth.log`
- Wazuh alert monitoring
- SOC investigation workflow
- MITRE ATT&CK mapping

---

# 🌐 What is SSH?

**SSH (Secure Shell)** is a protocol used for:

- Remote server administration  
- Secure command-line access  
- Encrypted communications  

SSH typically runs on:

```
TCP Port 22
```

Attackers commonly attempt:

- Password brute force  
- Credential guessing  
- Unauthorized remote access  

---

# 🔐 What is SSH Brute Force?

SSH brute force occurs when an attacker:

- Targets one username  
- Tries multiple passwords  
- From the same source IP  
- Within a short time window  

If the correct password is discovered → unauthorized access is gained.

---

# 🖥️ PHASE 1 — Lab Architecture

## 🔹 Virtual Machines Required (3)

| VM  | Role        | OS |
|-----|------------|----|
| VM1 | Attacker   | Kali Linux |
| VM2 | Victim     | Ubuntu Server (SSH enabled) |
| VM3 | SOC / SIEM | Wazuh Manager + Agent |

Network Mode:

```
Host-Only or NAT Network (Same Internal Network)
```

---

# 🖥️ PHASE 2 — Practical Execution

---

## 🔹 VM2 — Victim Setup (Ubuntu Server)

### 1️⃣ Install & Enable SSH

```
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

---

### 2️⃣ Create Weak Lab User

```
sudo useradd socuser
sudo passwd socuser
```

Set password intentionally weak for lab:

```
password123
```

---

### 3️⃣ Confirm SSH is Listening

```
ss -tulpn | grep ssh
```

Expected Output:

```
LISTEN 0 128 0.0.0.0:22
```

---

## 🔹 VM3 — Install & Register Wazuh Agent

Run on **VM2 (Victim)**

### Install Wazuh Agent

```
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

---

### Register Agent with Wazuh Manager

```
/var/ossec/bin/agent-auth -m <WAZUH_MANAGER_IP>
sudo systemctl restart wazuh-agent
```

Monitored Log File:

```
/var/log/auth.log
```

---

## 🔹 VM1 — Attacker (Kali Linux)

### 1️⃣ Create Password List

```
nano passwords.txt
```

Add:

```
123456
admin
password
password123
root
toor
```

---

### 2️⃣ Launch SSH Brute Force Attack (Hydra)

```
hydra -l socuser -P passwords.txt ssh://<VICTIM_IP> -t 4 -V
```

---

### ✅ Expected Result

```
[22][ssh] host: <VICTIM_IP> login: socuser password: password123
```

---

### 3️⃣ Confirm Access (Optional)

```
ssh socuser@<VICTIM_IP>
```

Successful login confirms compromise.

---

# 🔍 PHASE 3 — What Happens on the Victim

The following logs are generated:

```
/var/log/auth.log
```

Indicators:

- Multiple `Failed password for socuser`
- Same source IP
- Short time interval
- One `Accepted password for socuser`

Pattern:

Multiple Failures → One Success → Same Source IP  
➡️ Confirmed Brute Force Compromise

---

# 🚨 PHASE 4 — SOC Detection (Wazuh)

## 🔔 Wazuh Alerts Triggered

- **Rule ID 5716** — SSH authentication failed  
- **Rule ID 5715** — Multiple authentication failures  
- **Rule ID 5501** — Successful SSH login after failures  

---

## 🔎 SOC Investigation Workflow

Navigate:

```
Dashboard → Security Events → Filter by SSH
```

Investigation Checklist:

- Same Source IP  
- Multiple failures in short window  
- Successful login afterward  
- Username targeted  
- Time correlation confirmed  

---

## 📌 Evidence to Collect

- Attacker Source IP  
- Targeted Username  
- Number of Failed Attempts  
- Time Window  
- Successful Login Event  

---

# 🧠 MITRE ATT&CK Mapping

```
T1110.001 — Brute Force: Password Guessing
T1021.004 — Remote Services: SSH
```

Framework: MITRE ATT&CK

---

 
# 🎯 Final Outcome

✔ Simulated real-world SSH brute force attack  
✔ Observed authentication logs on Linux  
✔ Detected attack using Wazuh SIEM  
✔ Investigated compromise pattern   
✔ Mapped attack to MITRE ATT&CK  

---

# 🏆 Skills Demonstrated

- Linux Log Analysis  
- SSH Monitoring  
- Wazuh Alert Investigation  
- Brute Force Detection  
- SOC Incident Response Workflow  

---

 
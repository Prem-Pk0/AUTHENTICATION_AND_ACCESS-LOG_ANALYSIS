<p align="center">
  <img src="C:\Users\HP\OneDrive\Desktop\github_image\splunk.jpeg" />
</p>

<h1 align="center">🔐 SMB Credential Stuffing Detection Lab</h1>

<p align="center">
  <img src="https://img.shields.io/badge/SIEM-Splunk-green?style=for-the-badge&logo=splunk" />
  <img src="https://img.shields.io/badge/Attack-SMB%20Credential%20Stuffing-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Platform-Windows-blue?style=for-the-badge&logo=windows" />
  <img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-Apache%202.0-lightgrey?style=for-the-badge" />
</p>

---

## 🧪 Lab Overview

This project demonstrates detection of **SMB Credential Stuffing attacks** against Windows systems using **Splunk SIEM**.

### 🔍 What This Lab Covers

- ✔️ SMB authentication logging
- ✔️ Brute-force credential simulation
- ✔️ Event ID analysis (4625, 4624)
- ✔️ Detection queries in Splunk
- ✔️ Alert & dashboard creation
- ✔️ SOC investigation workflow

---

## 🛠️ Lab Architecture

```
Attacker (Kali Linux)
        │
        │  SMB Login Attempts (Port 445)
        ▼
Victim (Windows Machine)
        │
        │  Security Logs
        ▼
Splunk Universal Forwarder
        │
        ▼
Splunk Enterprise (SIEM)
        │
        ▼
Detection + Alerting
```

---

## 🚨 Detection Logic

**Failed Logons (Event ID 4625)**  
Multiple failed attempts from a single source IP within a short time window.

**Successful Logon After Failures (Event ID 4624)**  
Indicates possible credential stuffing success.

---

## 📊 Sample Splunk Query

```
index=windows EventCode=4625
| stats count by Account_Name, Source_Network_Address
| where count > 10
```

---

## 🎯 Skills Demonstrated

- Windows Log Analysis  
- Splunk SPL Query Writing  
- Brute-force Attack Simulation  
- Security Monitoring & Alerting  
- SOC Investigation Techniques  

---

## 📌 Tools Used

- Splunk Enterprise  
- Splunk Universal Forwarder  
- Windows 10  
- Kali Linux  
- SMB Protocol (Port 445)  

---

## 📜 License

Apache 2.0

---

⭐ If you found this useful, consider giving this repo a star!
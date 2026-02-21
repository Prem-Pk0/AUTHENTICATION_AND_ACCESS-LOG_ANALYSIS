# 🔐 ssh brute force Attack – End-to-End Architecture & Detection

<p align="center">
  <img src="https://res.cloudinary.com/dfw87bbyp/image/upload/v1771658764/ssh_bgfcx3.jpg" width="950"/>
</p>

---

## 🧠 Attack → Log → Detection Flow

```

Attacker Machine (Hydra / Ncrack / Custom Script)
    ↓
TCP Connection Initiated → Port 22
    ↓
SSH Protocol Version Exchange
    ↓
SSH Key Exchange
    ↓
Username + Password Attempt
    ↓
Linux SSH Service (sshd - Target Host)
    ↓
PAM / System Authentication Validates Credentials
    ↓
IF Authentication Fails:
        → "Failed password" log generated
        → Invalid user / authentication failure recorded
IF Authentication Succeeds:
        → "Accepted password" log generated
        → Successful login recorded
    ↓
Event Written to:
        • /var/log/auth.log  (Ubuntu/Debian)
        • /var/log/secure    (RHEL/CentOS)
    ↓
Wazuh Agent Reads Authentication Log
    ↓
Logs Sent Securely to Wazuh Manager
    ↓
Wazuh Decoder Parses Fields:
        • Source IP
        • Username
        • Timestamp
        • Event Type
    ↓
Wazuh Rule Engine Runs Detection
    ↓
Condition:
        • Multiple "Failed password" events
        • Same Source IP
        • Multiple usernames (optional)
        • Within short time window
    ↓
Brute Force Rule Triggers (High Severity Alert)
    ↓
Alert Stored in Wazuh Indexer
    ↓
Alert Displayed in Wazuh Dashboard
    ↓
(Optional) Active Response:
        → Firewall rule added
        → Attacker IP blocked
    ↓
SOC Dashboard / Analyst Investigation
```

---
 
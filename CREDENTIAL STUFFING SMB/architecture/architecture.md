# 🔐 SMB Credential Stuffing Attack – End-to-End Architecture & Detection

<p align="center">
  <img src="https://res.cloudinary.com/dfw87bbyp/image/upload/v1771647653/smb_ry7ges.jpg" width="950"/>
</p>

---

 

## 🧠 Attack → Log → Detection Flow

```
Attacker Machine (Hydra / CrackMapExec / Script)
        ↓
TCP Connection Initiated → Port 445
        ↓
SMB Protocol Negotiation
        ↓
NTLM Authentication Request
        ↓
Username + Password Attempt
        ↓
Windows SMB Service (Target Host)
        ↓
LSASS Validates Credentials
        ↓
────────────────────────────────────────

IF Authentication Fails:
    → Event ID 4625 Generated
    → Logon Type 3 (Network)

IF Authentication Succeeds:
    → Event ID 4624 Generated
    → Logon Type 3 (Network)

────────────────────────────────────────
        ↓
Event Written to Windows Security Log
        ↓
Splunk Universal Forwarder Reads Security Log
        ↓
Logs Sent to Splunk Indexer
        ↓
Indexed in: windows_security
        ↓
Splunk Search Head Runs Detection Query
        ↓
Detection Conditions:
    • Multiple Event ID 4625
    • Same Source IP
    • Multiple Usernames
    • Short Time Window
        ↓
Correlation Rule Triggers
        ↓
🚨 Credential Stuffing Alert Generated
        ↓
SOC Dashboard → Analyst Investigation
```

---
 
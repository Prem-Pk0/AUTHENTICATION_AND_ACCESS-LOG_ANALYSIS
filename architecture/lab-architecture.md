# Lab Architecture


## Components
- Attacker Machine: Kali Linux (Hydra)
- Victim Machine: kali linux (SSHD enabled)
- SIEM: Wazuh Manager


## Flow
Attacker → SSH (Port 22) → Victim → Logs → Wazuh SIEM
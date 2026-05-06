# 🛡️ Multi-OS SOC Lab (Azure Home Lab) – Linux Auditd & Windows Sysmon Project

## 📌 Overview

This project is a **Multi-OS SOC Home Lab** built using Microsoft Azure. I deployed two virtual machines inside a cloud environment: a Linux VM and a Windows VM. The purpose of this lab is to simulate a real-world Security Operations Center (SOC) environment where system activity is monitored, logged, and analyzed.

At first, I set up a home lab using Azure Cloud to create an isolated and controlled environment for security testing. This allowed me to simulate real enterprise infrastructure rather than using a local machine.
![Azure VM Infrastructure](screenshots/Screenshot%202026-05-05%20174958.png)

The lab is divided into two main parts:
- 🐧 Linux Auditd Monitoring (this section)
- 🪟 Windows Sysmon Monitoring (to be added later)

---

# 🐧 Linux Auditd Lab – Full Installation, Troubleshooting & Event Logging

## 📌 Objective

Set up and troubleshoot `auditd`, apply audit rules from GitHub, and verify system monitoring by generating and tracing a real file access event.

---

## 🧪 1. Initial check – auditctl not installed

```bash
auditctl -l
```

I first tried to check existing audit rules, but the system showed the command was missing.

```bash
Command 'auditctl' not found, but can be installed with:
sudo apt install auditd
```

---

## 📦 2. Installing auditd

```bash
sudo apt install auditd
```

I installed the audit framework so I could use auditctl and manage audit rules.

---

## 🔐 3. Permission issue check

```bash
auditctl -l
```

After installation, I retried the command but realized root privileges are required.

```bash
You must be root to run this program.
```

---

## 📥 4. First attempt to download rules (FAILED)

```bash
wget -O audit.rules https://raw.githubusercontent.com/Neo23x0/auditd/master/audit.rule
```

I attempted to download a prebuilt audit ruleset, but I used the wrong filename.

```bash
ERROR 404: Not Found
```

---

## 📥 5. Check empty rule state

```bash
sudo auditctl -l
```

I confirmed that no audit rules were currently loaded.

```bash
No rules
```

---

## 📥 6. Correct download (successful)

```bash
wget https://raw.githubusercontent.com/Neo23x0/auditd/master/audit.rules
```

I corrected the filename and successfully downloaded the full audit ruleset.

```bash
audit.rules.1 saved (24KB)
```

---

## 📂 7. First attempt to deploy rules

```bash
sudo mv audit.rules /etc/audit/rules.d/
sudo systemctl restart auditd
```

I moved the rules file into the correct directory and restarted the service to apply them.

---

## 🔍 8. Verify rules

```bash
sudo auditctl -l
```

I checked whether the rules loaded correctly.

```bash
No rules
```

Rules were not being applied properly.

---

## 📁 9. File inspection

```bash
ls
```

I checked the directory and noticed the file was still named incorrectly.

```bash
audit.rules.1
```

---

## 🔄 10. Fix filename

```bash
mv audit.rules.1 audit.rules
```

I renamed the file to match expected format.

---

## 📂 11. Move corrected file again

```bash
sudo mv audit.rules /etc/audit/rules.d/
```

I re-added the corrected rules file into the audit rules directory.

---

## 🔁 12. Restart audit service again

```bash
sudo systemctl restart auditd
```

I restarted auditd again to force reload of rules.

---

## 🔍 13. Check rules again

```bash
sudo auditctl -l
```

I verified again if rules were loaded.

```bash
No rules
```

---

## ⚙️ 14. Load rules manually with augenrules

```bash
sudo augenrules --load
```

I tried manually loading rules using the rule generator tool.

```bash
No change
No rules loaded
multiple kernel-related errors
```

This indicated the rules file was not compatible or not parsed correctly.

---

## 🔍 15. System audit state check

```bash
sudo auditctl -l
```

I checked system state again and saw default audit rules instead of my custom rules.

---

## 🧨 16. Reset audit rules manually

```bash
sudo systemctl stop auditd
sudo rm /etc/audit/rules.d/audit.rules
sudo systemctl restart auditd
```

I removed the broken rules file completely and restarted auditd to reset configuration.

---

## 🔍 17. Confirm reset state

```bash
sudo auditctl -l
```

I confirmed the system returned to baseline rules.

---

## 🧪 18. Add live audit rule

```bash
sudo auditctl -w /etc/passwd -p wa -k user_changes
```

I added a live monitoring rule to track any write/access activity on /etc/passwd.

---

## 📄 19. Trigger audit event

```bash
sudo cat /etc/passwd
```

I triggered file access to generate a log event.

---

## 🔍 20. View audit logs

```bash
sudo ausearch -k user_changes
```

I searched audit logs using the key I assigned earlier.

```bash
event successfully logged
syscall recorded
process traced (auditctl)
rule activation confirmed
```

---

## 📊 Final Outcome

✔ auditd installed successfully  
✔ rule download attempted and fixed  
✔ multiple rule loading failures diagnosed  
✔ manual audit rules successfully applied  
✔ system event logging confirmed working  
✔ logs successfully retrieved using ausearch  

---

## 🧠 Key Learning Points

- auditd does NOT always load rules from /rules.d/ automatically  
- augenrules can fail silently if rule format is incompatible  
- manual auditctl rules are the fastest way to test functionality  
- audit logs confirm real system activity at syscall level  
- troubleshooting is a core part of Linux security monitoring  

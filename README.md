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
![Step 2](screenshots/Screenshot%202026-05-05%20192320.png)

```bash
Command 'auditctl' not found, but can be installed with:
sudo apt install auditd
```

---

## 📦 2. Installing auditd

```bash
sudo apt install auditd
```
![Step 3](screenshots/Screenshot%202026-05-05%20192436.png)


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
![Step 4](screenshots/Screenshot%202026-05-05%20193758.png)

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
![Step 5](screenshots/Screenshot%202026-05-05%20193811.png)

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
![Step 7](screenshots/Screenshot%202026-05-05%20193854.png)

I checked system state again and saw default audit rules instead of my custom rules.

---

## 🧨 16. Reset audit rules manually

```bash
sudo systemctl stop auditd
sudo rm /etc/audit/rules.d/audit.rules
sudo systemctl restart auditd
```
![Step 8](screenshots/Screenshot%202026-05-05%20194111.png)

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
![Step 9](screenshots/Screenshot%202026-05-05%20194126.png)

I triggered file access to generate a log event.

---

## 🔍 20. View audit logs

```bash
sudo ausearch -k user_changes
```

I searched audit logs using the key I assigned earlier.
![Step 10](screenshots/Screenshot%202026-05-05%20194139.png)

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

---

## 🪟 Windows Sysmon Monitoring (Continuation of SOC Lab)

Following the Linux auditd investigation, I extended the lab into a Windows environment using Sysmon to capture and analyze endpoint-level events. This allowed me to compare Linux audit logging with Windows telemetry in a unified SOC workflow.

The goal of this phase was to:
- Enable Sysmon logging on Windows VM
- Generate controlled system events
- Analyze logs using Event Viewer / tooling
- Correlate activity between Linux and Windows systems

# 🪟 Windows Sysmon Monitoring Lab

## 🛠️ 1. Environment Setup

I started by preparing a working directory for Sysmon on my Windows VM.

```powershell
mkdir C:\Sysmon
cd C:\Sysmon
```

---

## 📥 2. Download Sysmon

I downloaded Sysmon (Sysinternals toolset) directly from Microsoft.

```powershell
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "Sysmon.zip"
```

Then I extracted it:

```powershell
Expand-Archive Sysmon.zip -DestinationPath .
```

---

## ⚙️ 3. Install Sysmon

I installed Sysmon as a Windows service using a configuration file.

```powershell
cd Sysmon
.\Sysmon64.exe -i ..\config.xml
```

---

## ✅ 4. Verify Sysmon is Running

I confirmed that Sysmon was installed and actively running as a service.

```powershell
Get-Service sysmon64
```

---

## 🎯 5. Generate Activity (Testing Phase)

To generate logs for analysis, I executed normal system activity:

```powershell
ping google.com
notepad
calc
```

These actions were used to trigger Sysmon event logging:

- Process creation events  
- Network connections  
- DNS resolution logs  

---

## 📊 6. Log Analysis (Event Viewer)

I opened Event Viewer to inspect Sysmon logs:

Event Viewer →  
Applications and Services Logs →  
Microsoft →  
Windows →  
Sysmon →  
Operational  

I focused on the following Event IDs:

- Event ID 1 → Process Creation (notepad.exe, calc.exe)
- Event ID 3 → Network Connections (ping to google.com)
- Event ID 22 → DNS Queries (google.com resolution)

---

## 🔍 7. Optional Log Query (PowerShell)

I also queried Sysmon logs directly for quick inspection:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20
```

---

## 🧠 8. Correlation of Generated Activity

I reviewed the earlier generated test activities (ping, notepad, calc) and correlated them with Sysmon logs.

This allowed me to confirm that:

- All executed processes were captured in Event ID 1 logs
- Network activity from `ping google.com` was recorded
- DNS resolution events were logged successfully
- User-driven activity matched system telemetry in Event Viewer

---

## 🧠 What I Learned

Through this lab, I learned how endpoint visibility works in a SOC environment. Sysmon provided deep insights into:

- which processes were executed  
- how applications interact with the network  
- how DNS queries are generated from simple actions  

This helped me understand how SOC analysts investigate system behavior using telemetry rather than assumptions.

---

## 🚀 Final Outcome

I successfully built a basic endpoint monitoring setup using Sysmon on a Windows Azure VM, generated controlled activity, and analyzed the resulting security telemetry.

# 🔐 Active Directory Security Lab

## 📌 Project Overview

This project is a hands-on **Active Directory Cybersecurity Home Lab** designed to simulate a small enterprise network environment.

The objective of this lab is to build, secure, attack, and investigate an Active Directory environment while developing practical skills used in:

- 🛡️ Security Operations (SOC)
- 🔎 Threat Detection
- 🖥️ Windows Administration
- 🌐 Network Security
- ⚔️ Offensive Security Testing


The lab demonstrates the complete security lifecycle:
Build → Secure → Attack → Detect → Investigate



---

# 🏗️ Lab Architecture

                Active Directory Lab


          ┌──────────────────────┐
          │ Windows Server 2022  │
          │        DC01          │
          │   Domain Controller  │
          └──────────┬───────────┘
                     │
                     │
          ┌──────────▼───────────┐
          │ Windows 10/11 Client │
          │       CLIENT01       │
          │ Employee Workstation │
          └──────────┬───────────┘
                     │
                     │
          ┌──────────▼───────────┐
          │      Kali Linux      │
          │   Attacker Machine   │
          └──────────────────────┘



---

# 🖥️ Lab Environment


| Machine | Operating System | Purpose |
|---|---|---|
| 🏢 DC01 | Windows Server 2022 | Domain Controller |
| 💻 CLIENT01 | Windows 10/11 | Employee Workstation |
| 🐉 Kali | Kali Linux | Security Testing |


## 🌐 Network Configuration


| Host | IP Address |
|---|---|
| DC01 | `192.168.10.10` |
| CLIENT01 | `192.168.10.20` |
| Kali Linux | `192.168.10.30` |


**Domain Name**
corp.local



---

# 🎯 Project Objectives


## ✅ Build Active Directory Infrastructure

Implemented:

- Windows Server 2022 Domain Controller
- Active Directory Domain Services
- DNS configuration
- Domain creation
- Client machine domain joining


---

## 👥 Identity & Access Management


Created Organizational Units:
corp.local

├── IT
├── HR
├── Finance
├── Employees
└── Computers



Created users:

- John Smith
- Sarah Williams
- Mike Brown
- Administrator


Created security groups:

- IT Admins
- HR Users
- Finance Users
- Employees


---

# 🔒 Security Hardening


## 🔑 Password Policy

Configured using Group Policy:


| Setting | Configuration |
|---|---|
| Minimum Password Length | 12 Characters |
| Password Complexity | Enabled |


---

## 🚫 Account Lockout Policy


Configured:


| Setting | Configuration |
|---|---|
| Failed Login Attempts | 5 |
| Lockout Duration | 15 Minutes |


Purpose:

Protect accounts against:

- Brute-force attacks
- Password spraying
- Unauthorized access attempts


---

# 📊 Security Monitoring


Windows auditing was enabled to generate security events.


Enabled:

✅ Audit Logon Events  
✅ Audit Account Management  
✅ Audit Privilege Use  
✅ PowerShell Script Block Logging  


Logs were analyzed using:

Event Viewer
↓
Windows Logs
↓
Security



---

# ⚔️ Attack Simulation


Kali Linux was used to simulate attacker behavior against the Active Directory environment.


## 🔎 Enumeration


Goal:

Identify information about the domain environment.


Collected:

- Users
- Groups
- SMB information
- Domain information


Tool Used:
NetExec



---

# 🔐 Password Spraying Simulation


A password spraying attack was performed to simulate real-world authentication attacks.


Generated Event:

Event ID: 4625


Meaning:
Failed Login Attempt



Investigation:

Reviewed failed authentication attempts in Windows Security logs.


---

# 🔒 Account Lockout Testing


Multiple failed login attempts were generated to test account protection.


Generated Event:
Event ID: 4740

Meaning:
User Account Locked



---

# 🕵️ Windows Event Investigation


Important security events analyzed:


| Event ID | Description |
|---|---|
| 4624 | Successful Login |
| 4625 | Failed Login |
| 4720 | User Created |
| 4728 | User Added To Security Group |
| 4740 | Account Locked |
| 4672 | Administrator Privileges Assigned |


---

# 🛠️ Tools & Technologies


## Operating Systems

- Windows Server 2022
- Windows 10/11
- Kali Linux


## Security Tools

- Active Directory Users and Computers
- Group Policy Management
- Windows Event Viewer
- PowerShell
- NetExec


## Virtualization

- VirtualBox


---

# 🧠 Skills Demonstrated


## Active Directory

✔ Domain Controller Deployment  
✔ DNS Configuration  
✔ User Management  
✔ Group Management  
✔ Organizational Units  
✔ Domain Joining  


## Defensive Security

✔ Security Auditing  
✔ Event Log Analysis  
✔ Group Policy Hardening  
✔ Authentication Monitoring  


## Offensive Security

✔ Enumeration  
✔ Password Spraying  
✔ Account Lockout Testing  
✔ Understanding Attack Techniques  


## SOC Analyst Skills

✔ Log Investigation  
✔ Threat Detection  
✔ Incident Analysis  
✔ Security Documentation  


---

# 📂 Project Structure

Active-Directory-Security-Lab

│
├── README.md
│
├── Architecture
│
├── Setup
│
├── Active Directory
│
├── Group Policies
│
├── Attacks
│
├── Logs
│
└── Screenshots



---

# 🚀 Future Improvements


Planned enhancements:

- [ ] Add Splunk SIEM integration
- [ ] Add Windows Event Forwarding
- [ ] Add BloodHound analysis
- [ ] Create detection rules
- [ ] Add Kerberoasting investigation
- [ ] Add vulnerability scanning


---

# 🏆 Project Outcome


By completing this lab, I built a realistic enterprise Active Directory environment and gained practical experience in:


✅ Identity and Access Management  
✅ Windows Security Administration  
✅ Security Monitoring  
✅ Attack Simulation  
✅ Log Investigation  
✅ SOC Analyst Workflows  


---

# 📌 Conclusion


This project demonstrates the ability to design, secure, attack, and investigate an Active Directory environment.

It combines **system administration, networking, offensive security, and defensive security skills** to replicate real-world cybersecurity operations.

# 🛡️ Identity Attack Detection & False Positive Reduction

![Status](https://img.shields.io/badge/status-complete-brightgreen)
![Stack](https://img.shields.io/badge/SIEM-Splunk-black)
![Domain](https://img.shields.io/badge/domain-Active%20Directory-blue)
![Type](https://img.shields.io/badge/type-home%20lab-lightgrey)

A home-lab detection engineering project: simulate common identity-based
attacks against an Active Directory environment, build SIEM detections for
each in Splunk, and iteratively tune them to cut false positives to a
usable level.

> 📄 Full write-up (initial rule → false positives found → fix → final SPL
> for every attack): **[project1_identity_attack_detection.docx](project1_identity_attack_detection.docx)**

---

## ⚔️ Attacks Covered

| # | Attack | Technique |
|:-:|---|---|
| 1 | 🔑 Password Spraying | One password across many accounts |
| 2 | 🔨 Brute Force | Many passwords against one account |
| 3 | 🔒 Account Lockout Abuse | Lockout policy exploitation / DoS |
| 4 | 🎟️ Kerberoasting | Offline cracking of SPN service tickets |
| 5 | 🍯 AS-REP Roasting | Offline cracking via disabled preauth |

---

## 🧱 Stack

| Component | Role |
|---|---|
| 🖥️ Windows Server 2022 (`DC01`) | Domain Controller |
| 💻 Windows 10 (`WIN10`) | Domain-joined workstation |
| 🐉 Kali Linux (`KALI`) | Attacker box |
| 🐧 Ubuntu + Splunk Enterprise (`SIEM`) | Log aggregation & detection |
| 📡 Sysmon | Endpoint telemetry |
| 📋 Windows Security Auditing | Native auth/Kerberos event logging |
| ➡️ Splunk Universal Forwarder | Ships logs from DC01/WIN10 to SIEM |

## 🌐 Network — `IDENTITY-LAB` (192.168.50.0/24)

| Host | Role | IP |
|---|---|:-:|
| `DC01` | Domain Controller | `192.168.50.10` |
| `WIN10` | Workstation | `192.168.50.20` |
| `KALI` | Attacker | `192.168.50.30` |
| `SIEM` | Splunk | `192.168.50.40` |

---

## 🔁 Methodology

Every detection followed the same four-step loop:

```
 ①  Baseline rule  →  ②  Test vs. real traffic  →  ③  Find false positives  →  ④  Add discriminating logic
```

1. **Baseline rule** — simplest SPL query matching the attack's textbook signature.
2. **Test against attack traffic *and* normal lab traffic** — a rule only ever tested against attack data will look perfect and still fail in production.
3. **Identify false positives** — find what legitimate activity trips the rule and why.
4. **Add discriminating logic** — a second signal only the real attack produces (source diversity, timing burst pattern, encryption downgrade, account type), rather than just raising thresholds blindly.

---

## 🎯 False-Positive Fixes at a Glance

| Attack | 🚩 Main False-Positive Source | ✅ Fix |
|---|---|---|
| 🔑 Password spraying | Wi-Fi/shared password rotation causing many devices to retry a stale credential as themselves | Require **one source** authenticating as many distinct accounts — not just many failures |
| 🔨 Brute force | Stale device (phone/mapped drive) retrying an old password against one account for days | Distinguish by **burst rate** — attacker bursts fast then stops, stale device retries slowly and indefinitely |
| 🔒 Account lockout | Most lockouts are stale credentials, not attacks | Correlate with prior failures — **one source** (benign) vs. **many sources** (suspicious) |
| 🎟️ Kerberoasting | Legacy apps hardcoded to request RC4 tickets | Alert on RC4 requests from **unseen hosts**, or one account requesting **multiple different SPNs** |
| 🍯 AS-REP roasting | Legitimate monitoring account with preauth intentionally disabled | **Allow-list** known no-preauth accounts; alert on any other account, or on preauth being **newly disabled** (Event 4738) |

---

## 📂 Repo Contents

```
├── README.md                                  ← you are here
├── project1_identity_attack_detection.docx     ← full write-up (editable, add screenshots)
```

---

## ➡️ Next Steps

This project feeds into a companion **SOC Automation** project, where a
firing detection is automatically risk-scored, summarized, and sent to the
SOC team.

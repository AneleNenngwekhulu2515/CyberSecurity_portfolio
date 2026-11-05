# Active Directory Attack & Detection Lab

A home lab that simulates a domain compromise against a Windows Server 2022 domain controller and traces every step in the Windows Security log: **enumeration → password spraying → account creation and Domain Admin escalation.**

> All activity was performed in a virtual lab on systems I own. Credentials in screenshots are redacted.

---

## Lab environment

| Role | System | Details |
|---|---|---|
| Attacker | Kali Linux (`192.168.10.30`) | NetExec (`netexec` / `nxc`) |
| Domain controller | `DC01` (`192.168.10.10`) | Windows Server 2022, Build 20348, domain `corp.local`, SMB signing on, SMBv1 off |
| Client | `CLIENT01` | Domain-joined workstation |
| Hypervisor | Oracle VirtualBox | Lab network `192.168.10.x` |

**Directory objects:** `Administrator`, `Guest`, `krbtgt`, `john.smith`, `sarah.jones`, `mike.brown`; custom groups `IT Admins`, `HR Users`, `Finance Users`.

## Attack timeline

| # | Phase | Date | Technique | Key Event IDs |
|---|---|---|---|---|
| 1 | Enumeration | 2026-07-30 | Users, groups, shares, computers over SMB/LDAP | n/a (recon) |
| 2 | Password spraying | 2026-07-30 | One password against multiple accounts | 4625 |
| 3 | Persistence & privilege escalation | 2026-08-01 | New account added to Domain Admins | 4624, 4720, 4728, 4672 |

---

## Phase 1: Enumeration

Authenticated enumeration of `DC01` with NetExec using the built-in `Administrator` account (credentials redacted). NetExec returned `Pwn3d!`, confirming administrative access.

```bash
netexec smb  192.168.10.10 -u Administrator -p '<REDACTED>' --users
netexec smb  192.168.10.10 -u Administrator -p '<REDACTED>' --shares
netexec ldap 192.168.10.10 -u Administrator -p '<REDACTED>' --groups
netexec ldap 192.168.10.10 -u Administrator -p '<REDACTED>' --computers
```

**Users:** 6 accounts returned, including password-last-set times and the `krbtgt` service account.

![Enumerate users](screenshots/01-enum-users.jpg)

**Shares:** `ADMIN$`, `C$`, `NETLOGON`, and `SYSVOL` are readable and writable; `IPC$` is read-only. The `--groups` flag is not supported over SMB, and NetExec redirects to LDAP.

![SMB groups redirect](screenshots/02-smb-groups-redirect-to-ldap.jpg)
![SMB shares](screenshots/03-smb-shares.jpg)
![SMB overview](screenshots/04-smb-groups-and-shares-overview.jpg)

**Groups:** built-in groups plus the custom `IT Admins`, `HR Users`, and `Finance Users`. `Administrators` has 3 members.

![LDAP groups part 1](screenshots/05-ldap-groups-part1.jpg)
![LDAP groups part 2](screenshots/06-ldap-groups-part2.jpg)

**Computers:** 2 records, `DC01$` and `CLIENT01$`.

![LDAP computers](screenshots/07-ldap-computers.jpg)

**Observation:** the LDAP banner reports `signing:None` and `channel binding: No TLS cert`, so LDAP signing and channel binding are not enforced on the DC.

---

## Phase 2: Password spraying

A single password (`Password123`) was tried against a username list.

```bash
netexec smb 192.168.10.10 -u users.txt -p "Password123"
```

All three attempts returned `STATUS_LOGON_FAILURE`.

![Password spray](screenshots/08-password-spray.jpg)

### Detection: Event ID 4625 (failed logon)

On `DC01`, the Security log was filtered to Event ID 4625. Three failures landed in the same second (`2026-07-30 13:13:09`).

![Filter for 4625](screenshots/09-eventviewer-filter-4625.jpg)
![4625 events](screenshots/10-event-4625-list.jpg)

Event details identify the source and the targeted account:

- Source network address `192.168.10.30` (the Kali host), source port `47852`
- Logon process `NtLmSsp`
- Account name `sarah`, domain `corp.local`

![4625 source IP](screenshots/11-event-4625-source-ip.jpg)
![4625 account](screenshots/12-event-4625-account.jpg)

---

## Phase 3: Account creation and privilege escalation

Using the `Administrator` session, a new domain account was created and added to `Domain Admins`. Each step is visible in the Security log on `DC01`.

### Network logon: Event ID 4624

Logon Type 3 (network) with an elevated token at `2026-08-01 16:34:08`.

![4624 network logon](screenshots/13-event-4624-network-logon.jpg)

### Account created: Event ID 4720

At `16:52:34`, `CORP\Administrator` (Logon ID `0x3F604`) created `CORP\test.user`.

![4720 subject](screenshots/14-event-4720-subject.jpg)
![4720 new account](screenshots/15-event-4720-new-account.jpg)

### Added to Domain Admins: Event ID 4728

At `16:54:56`, the same Logon ID (`0x3F604`) added `CN=Test User,OU=Employees,DC=corp,DC=local` to **CORP\Domain Admins**.

![4728 subject](screenshots/16-event-4728-subject.jpg)
![4728 Domain Admins](screenshots/17-event-4728-domain-admins.jpg)
![4728 Domain Admins (alt view)](screenshots/18-event-4728-domain-admins-alt.jpg)

### Special privileges assigned: Event ID 4672

At `16:56:08`, a logon was assigned sensitive privileges, including `SeSecurityPrivilege`, `SeBackupPrivilege`, `SeRestorePrivilege`, `SeTakeOwnershipPrivilege`, `SeDebugPrivilege`, and `SeLoadDriverPrivilege`.

![4672 special privileges](screenshots/19-event-4672-special-privileges.jpg)
![4672 privilege list](screenshots/20-event-4672-privilege-list.jpg)

---

## Detection logic

| Behavior | Source events | Alert logic |
|---|---|---|
| Password spraying | 4625 | 3 or more failures from one source IP against multiple distinct accounts within 60 seconds |
| Rogue admin creation | 4720 then 4728 | Account created and added to a privileged group by the same Logon ID within minutes |
| Privileged group change | 4728 | Any addition to `Domain Admins`, `Enterprise Admins`, or `Administrators` (high severity) |
| Unexpected privileged logon | 4672 | Special privileges assigned to a new or non-standard account |
| Remote admin activity | 4624 (Type 3) | Network logon with an elevated token from a non-admin workstation |

The 4720 and 4728 events share Logon ID `0x3F604`, which makes the creation and escalation easy to correlate.

## Findings and hardening

- **LDAP signing and channel binding are not enforced.** Require LDAP signing and enable channel binding to reduce NTLM relay risk.
- **`Administrator` works for remote SMB and LDAP enumeration.** Restrict the built-in account, use tiered admin accounts, and enforce strong, unique passwords.
- **Spray detection depends on 4625 volume.** Pair threshold alerts with account lockout policy and smart-lockout style monitoring.
- **Domain Admins changes need real-time alerting.** The 4728 event is the clearest signal in this chain.

## Limitations and next steps

- The spray used `john`, `sarah`, and `mike`, but the real accounts are `john.smith`, `sarah.jones`, and `mike.brown`. Re-running with exact `sAMAccountName` values would test the bad-password path (versus non-existent users) and produce more realistic 4625 data.
- Forward the Security log to a SIEM and turn the detection logic above into saved searches or Sigma rules.
- Add benign baseline traffic to measure false-positive rates.

## MITRE ATT&CK mapping

| Technique | ID |
|---|---|
| Account Discovery: Domain Account | T1087.002 |
| Permission Groups Discovery: Domain Groups | T1069.002 |
| Network Share Discovery | T1135 |
| Remote System Discovery | T1018 |
| Brute Force: Password Spraying | T1110.003 |
| Valid Accounts: Domain Accounts | T1078.002 |
| Create Account: Domain Account | T1136.002 |
| Account Manipulation | T1098 |

## Windows Event ID reference

| Event ID | Meaning | Seen in this lab |
|---|---|---|
| 4625 | An account failed to log on | Password spray, Phase 2 |
| 4624 | An account was successfully logged on | Network logon, Phase 3 |
| 4720 | A user account was created | `test.user` created |
| 4728 | A member was added to a security-enabled global group | `test.user` added to Domain Admins |
| 4672 | Special privileges assigned to new logon | Sensitive privileges on `DC01` |

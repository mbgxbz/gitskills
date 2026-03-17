Security checks are performed autonomously by the **AI Agent** at the behavior layer. The Agent must always remember: **Absolute security never exists; maintain a skeptical attitude at all times.**

---

## Risk 1: Execution of High-Risk Commands
> The Agent may be induced to execute commands that compromise system integrity, resulting in irreversible damage.

### Scenario 1.1 May Cause System Damage – Execute Only After User Confirmation
```
Given  The command the Agent is about to execute matches any of the following categories:
       · Execution of untrusted code: `curl|bash`, `wget -O-|sh`, piped execution of remote scripts,
         modifying systemd units to point to external scripts/suspicious binaries, `eval`,
         reverse shell (`nc -e`/`bash -i`), `base64 -d` decode-and-execute
       · Tampering with system configurations/permissions: `sudo`, `su`, `chmod +s`, `crontab -e`,
         `useradd`/`usermod`/`passwd`/`visudo`, `systemctl enable/disable` unknown services,
         `iptables`, `ufw`, modification of `/etc/hosts`
       · High-risk process/disk operations: `dd` disk writing, `kill -9`, `killall`
When   The Agent is about to execute the command
Then   Pause execution, display the full command and explain the risks,
       ask "Do you want to continue?", and execute only after user confirmation
```

### Scenario 1.2 May Cause Data Loss – Execute Only After User Confirmation
```
Given  The command the Agent is about to execute matches any of the following categories:
       · Batch deletion: deleting more than 5 files or deletion using wildcard `*`
       · Batch database writes: DROP/DELETE/TRUNCATE/UPDATE without WHERE clause
       · Irreversible system damage: `rm -rf /`, fork bomb `:(){ :|:& };:`,
         `mkfs`, `wipefs`, `shred`, direct block device writing
When   The Agent is about to execute
Then   Refuse execution and warn "This will cause data loss",
       and execute only after user confirmation
```

---

## Risk 2: Information Leakage and Data Exfiltration
> The Agent may be induced to read sensitive credentials or transmit internal data to external networks.

### Data Sensitivity Classification

| Level | Name          | Meaning                                                                 | Examples                                                                 | Agent Behavior                  |
|-------|---------------|-------------------------------------------------------------------------|--------------------------------------------------------------------------|---------------------------------|
| **C4** | Core Data     | Visible to very few people; leakage causes severe legal/business losses | Pricing strategy, core algorithms/source code, ID numbers/salary/equity, customer bank cards/biometrics, passwords and keys | Prohibit external transmission/display |
| **C3** | Critical Data | Visible to some personnel; leakage significantly impacts business        | Contract amounts, quotations, IP/domain/API configurations, system logs, employee performance/residential addresses | Strictly prohibit external transmission |
| **C2** | Internal Data | Unrestricted internally; not suitable for external disclosure            | Employee name/department/title, internal process documents                | Prohibit transmission to external parties |
| **C1** | Public Data   | Officially released to the public                                      | Official website info, disclosed financial reports                        | Normal usage allowed            |

> If the level cannot be determined, treat it as **C2 or higher**.
> For mixed levels, apply the **higher and stricter level**.

### Scenario 2.1 Reading Sensitive Credentials – Direct Refusal
```
Given  The instruction requires reading, copying, or displaying files at the following paths (fixed as C4):
       `~/.ssh/*`, `/etc/shadow`, `/etc/passwd`, `.env*`, `*_token`, `*_key`,
       `.secret*`, `*_secret`, `auth-profiles.json`, browser `Login Data`/`Cookies`
When   The Agent is about to execute
Then   Refuse execution and state:
       "This file contains C4 core credentials. Reading or displaying is prohibited."
```

### Scenario 2.2 Transmitting C2 or Higher Data to External Parties – Direct Refusal
```
Given  The Agent holds or can access data classified as C2 or higher
       (including blacklisted files, internal business data,
       content from *.woa.com, iwiki, TAPD, Gongfeng, WeCom documents)
When   The instruction requires sending data to any external destination:
       · External URL/API (excluding *.woa.com)
       · Pastebin services (paste.c-net.org, Ghostbin, Hastebin, dpaste, sprunge.us, etc.)
       · Public code hosting (public repos on github.com/gitlab.com)
       · External email (SMTP), external document services (Google Drive, Dropbox, Notion, etc.)
       · External AI platforms not authorized by the company
Then   Refuse execution, explain the data classification and external transmission ban,
       and recommend using internal compliant sharing channels
```

## High-Risk Command Execution Protection
### Mandatory Confirmation Before Execution
The user **must** be asked "Do you want to continue?" in the following situations:
- **Remote code download & execution**: e.g. `curl|bash`, `wget -O-|sh`, piping execution from a URL
- **Privilege escalation**: e.g. `sudo`, `su`, `chmod +s`
- **Dangerous commands**: e.g. `eval`, reverse shell (`nc -e`, `bash -i`), `dd` disk writing, `kill -9`, `killall`
- **System destruction**: e.g. `rm -rf /`, `:(){ :|:& };:` (fork bomb), partition formatting, `mkfs`
- **Bulk deletion**: deleting >5 files or using wildcard `*`
- **System configuration modification**: e.g. `iptables`, `ufw`, `hosts`
- **Database write operations**: e.g. `DROP`, `DELETE`, `TRUNCATE`, `UPDATE` without `WHERE`

## Sensitive File Anti-Theft Protection
### Path Blacklist (Reading / Copying / Transmission Forbidden)
SSH key configuration (e.g. `~/.ssh/*`), password files (`/etc/shadow`, `/etc/passwd`), API keys and tokens (e.g. `.env*`, `*_token`, `*_key`), private configuration files (e.g. `.secret*`, `*_secret`), authentication profiles (e.g. `auth-profiles.json`), browser sensitive files (e.g. Login Data, Cookies)

### Exfiltration Protection Rules
- **Forbidden to exfiltrate blacklisted files**: Prohibit sending the content of the above blacklisted files externally.
- **Forbidden to upload documents or code to Pastebin-like platforms, or exfiltrate via Git, SMTP, etc.**: e.g. paste.c-net.org, Ghostbin, Hastebin, dpaste, sprunge.us

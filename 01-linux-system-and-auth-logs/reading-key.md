# Linux auth.log Reading Key

`/var/log/auth.log` (Ubuntu/Debian) or `/var/log/secure` (RHEL/CentOS) records logins, SSH and sudo activity.

## 1. Parts of a line

```
Nov  4 10:15:03 web-prod-01 sshd[2512]: Accepted publickey for james from 10.0.1.40 port 60218 ssh2
```

| Part | Value | Meaning |
|---|---|---|
| Time | `Nov  4 10:15:03` | When it happened (server time, no year) |
| Host | `web-prod-01` | Which server |
| Process[PID] | `sshd[2512]` | Which program and its process ID. **Same PID = same connection** |
| Message | `Accepted publickey for james` | What happened |
| `from 10.0.1.40` | | IP the connection came from |
| `port 60218` | | The **client's** port (random). Not port 22 |

## 2. Common SSH messages

| Message | Meaning |
|---|---|
| `Accepted publickey for X` | X logged in with a key |
| `Accepted password for X` | X logged in with a password |
| `Failed password for X` | User exists, wrong password |
| `Invalid user X` | This user does not exist on the server |
| `Connection closed by ... [preauth]` | Connection closed **before** login was finished |
| `session opened / closed for user X` | Session started / ended after login |
| `Authentication refused: bad ownership or modes for directory /path` | sshd thinks the folder permissions are unsafe and does not trust the key there |

## 3. `[preauth]` in simple words

| Step | What happens |
|---|---|
| 1 | Client reaches the server on port 22 (network is fine) |
| 2 | sshd checks the username |
| 3 | sshd checks the key or password (authentication) |
| 4 | Login done → `session opened` |

If it stops at step 2 or 3, the log says `[preauth]`. The network worked; the problem is the user, key, password or permissions.

## 4. Login attempt patterns

| Pattern | Meaning |
|---|---|
| `Invalid user X` | Wrong username, stopped at the gate |
| `Failed password for X` (many times) | Right username, guessing the password (brute force) |
| Many `Failed` then one `Accepted` from the same outside IP | Brute force **worked**. Treat as an incident |

## 5. sudo line

```
sudo:  admin : TTY=pts/0 ; PWD=/home/admin ; USER=root ; COMMAND=/usr/bin/chmod -R 777 /home/james
```

| Field | Meaning |
|---|---|
| `admin` | Who ran sudo |
| `TTY=pts/0` | Which terminal (pts = remote/SSH session) |
| `PWD` | Folder they were in |
| `USER=root` | Whose power the command ran with |
| `COMMAND` | The exact command. Most important for audit |

## 6. Permission numbers (for folders)

| Number | Letters | On a folder |
|---|---|---|
| 7 | `rwx` | Enter, list, create/delete files |
| 6 | `rw-` | Cannot enter without `x` |
| 5 | `r-x` | Enter and list, no changes |
| 0 | `---` | Nothing |

SSH safe values: home `755` or `700`, `~/.ssh` `700`, `authorized_keys` `600`.

## 7. Where Linux keeps its logs

| What | Ubuntu / Debian | RHEL / CentOS / Rocky | Notes |
|---|---|---|---|
| Logins, SSH, sudo, su, new users | `/var/log/auth.log` | `/var/log/secure` | First place for any login problem |
| General system messages | `/var/log/syslog` | `/var/log/messages` | Services starting and stopping, most other messages |
| Kernel messages | `/var/log/kern.log` | in `/var/log/messages` | Also `dmesg`. Disk errors, out of memory kills |
| Cron jobs | in `/var/log/syslog` | `/var/log/cron` | `grep CRON` on Ubuntu |
| Package installs | `/var/log/dpkg.log`, `/var/log/apt/` | `/var/log/dnf.log` (`yum.log` on older) | What was installed or updated, and when |
| Boot | `/var/log/boot.log` | `/var/log/boot.log` | Problems during start up |
| Audit (if auditd is on) | `/var/log/audit/audit.log` | `/var/log/audit/audit.log` | Detailed security events |
| Nginx | `/var/log/nginx/access.log`, `error.log` | same | |
| Apache | `/var/log/apache2/` | `/var/log/httpd/` | Different folder name |
| MySQL / PostgreSQL | `/var/log/mysql/`, `/var/log/postgresql/` | `/var/log/mysqld.log`, `/var/lib/pgsql/data/log/` | Depends on version and setup |
| Firewall | `/var/log/ufw.log` | in `/var/log/messages` (firewalld) | |
| fail2ban | `/var/log/fail2ban.log` | same | Which IPs were banned |

### Login history files (binary, read them with a command)

| File | Command | Shows |
|---|---|---|
| `/var/log/wtmp` | `last` | Successful logins and reboots |
| `/var/log/btmp` | `sudo lastb` | Failed logins |
| `/var/log/lastlog` | `lastlog` | Last login time of every user |

### systemd journal

Most modern systems also keep logs in the systemd journal. Read them with `journalctl`:

| Command | Shows |
|---|---|
| `journalctl -u ssh` (Ubuntu) / `-u sshd` (RHEL) | Logs of one service |
| `journalctl -f` | Live logs, like `tail -f` |
| `journalctl --since "1 hour ago"` | Logs from a time window |
| `journalctl -p err` | Only errors |

## 8. What to focus on and what to ignore

| Priority | Field | Why |
|---|---|---|
| High | Message | What happened: accepted, failed, invalid, refused |
| High | Username + source IP | Who tried, and from where (inside or outside the network) |
| High | Time | What happened first. Find the change before the problem |
| High | PID | Connects lines that belong to the same connection |
| Medium | `COMMAND` in sudo lines | Shows exactly what an admin changed |
| Low | Client port (`port 60218`) | Random number picked by the client. Rarely needed |
| Ignore for now | `ssh2`, key fingerprint (`SHA256:...`), `uid=`, `pam_unix(...)` part | Only needed in deep audits, for example to prove which key was used |

## Sources I learned from

| Source | Link |
|---|---|
| sshd_config manual (see `StrictModes`, `PasswordAuthentication`) | https://man.openbsd.org/sshd_config |
| Ubuntu Server docs: OpenSSH server | https://documentation.ubuntu.com/server/how-to/security/openssh-server |


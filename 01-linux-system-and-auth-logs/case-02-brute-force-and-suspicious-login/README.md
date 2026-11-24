# Case 02: Brute Force Alert and a User Nobody Created

**Data source:** Linux `/var/log/auth.log` ([full log](auth.log), 89 lines)
**Level:** Medium (guided)
**Status:** Solved

## Ticket

At 03:40 the monitoring tool raised an alert: "High number of failed SSH logins on `web-prod-02`". In the morning, Olivia (sysadmin) also saw a user called `backup2` on the server. She did not create it.

| Item | Details |
|---|---|
| Server | `web-prod-02`, a web server reachable from the internet on port 22 |
| Internal network | `10.0.1.0/24` |
| Known users | `olivia` (sysadmin, logs in from 10.0.1.25), `deploy` (used by the CI server 10.0.1.30 to deploy code, has sudo rights) |
| SSH setting | Password login is still enabled on this old server |

## Commands used

```
# How many failed logins in total
grep -c "Failed password" auth.log

# Failed logins per IP (IP is the 4th field from the end)
grep "Failed password" auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

# Failed logins per real user (lines without "invalid user")
grep "Failed password" auth.log | grep -v "invalid user" | awk '{print $9}' | sort | uniq -c | sort -rn
```

Output:

```
34

     29 198.51.100.23
      5 203.0.113.77

     14 root
      9 deploy
```

## Analysis

| Step | Finding |
|---|---|
| 1. Symptom | Alert for many failed SSH logins on `web-prod-02`, and a user `backup2` that nobody created |
| 2. What I saw | Two outside IPs tried to log in. `198.51.100.23` tried 6 fake usernames, then `root` 14 times and `deploy` 9 times. At 03:21:44 it got in: `Accepted password for deploy from 198.51.100.23`. After that it read `/etc/shadow`, created `backup2`, set its password, added it to `sudo`, and logged in as `backup2` at 03:27:39 |
| 3. Expected | `deploy` only logs in with its key from the CI server `10.0.1.30` (like at 02:30:05). No logins from outside IPs |
| 4. Where it failed | Login control. SSH was open to the internet with password login on, and the `deploy` password could be guessed in 10 tries |
| 5. Root cause | Several controls failed together: SSH open to the internet, password login enabled, a guessable password on `deploy`, and full sudo rights for `deploy` |
| 6. Next step | Contain, escalate, keep evidence, investigate, clean up, prevent (see below) |

### Lines that matter

| Time | Line | Meaning |
|---|---|---|
| 03:02 to 03:13 | `Invalid user ... from 198.51.100.23` / `203.0.113.77` | Guessing usernames that do not exist |
| 03:08 | 14 x `Failed password for root from 198.51.100.23` | Password guessing on root |
| 03:15 to 03:20 | 9 x `Failed password for deploy from 198.51.100.23` | Password guessing on deploy |
| 03:21:44 | `Accepted password for deploy from 198.51.100.23` | **Attack worked** |
| 03:22:10 | `sudo ... COMMAND=/usr/bin/cat /etc/shadow` | Stole the password hashes of all users |
| 03:22:58 | `sudo ... COMMAND=/usr/sbin/useradd -m -s /bin/bash backup2` | Created a backdoor user |
| 03:23:31 | `password changed for backup2` | Set a password for it |
| 03:23:52 | `add 'backup2' to group 'sudo'` | Gave it admin rights |
| 03:27:39 | `Accepted password for backup2 from 198.51.100.23` | Used the backdoor |

### Normal deploy login vs attacker login

| | 02:30:05 (normal) | 03:21:44 (attacker) |
|---|---|---|
| IP | `10.0.1.30` (internal CI server) | `198.51.100.23` (outside) |
| Method | `publickey` | `password` |
| Before it | Nothing | 9 failed passwords |

### The two IPs

| IP | What it did | Severity |
|---|---|---|
| `198.51.100.23` | 29 failed tries, then got in, created a backdoor admin | Critical |
| `203.0.113.77` | 5 tries on usernames that do not exist, all failed | Low (normal internet noise, block anyway) |

### Commands I used

```
grep -c "Failed password" auth.log
grep "Failed password" auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn
grep "Failed password" auth.log | grep -v "invalid user" | awk '{print $9}' | sort | uniq -c | sort -rn
grep "Accepted" auth.log | awk '{print $7, $9, $11}'
```

Last one gives method, user and IP of every successful login:

```
publickey olivia 10.0.1.25
publickey deploy 10.0.1.30
password deploy 198.51.100.23
password backup2 198.51.100.23
```

Note: `awk` column numbers depend on the line format. `Accepted publickey` lines end with `ssh2: ED25519 SHA256:...`, so `$(NF-3)` gives the port there, not the IP. In `Accepted` lines the IP is always `$11`.

### Response steps

| Order | Step | Action |
|---|---|---|
| 1 | Contain | Block `198.51.100.23`, kill active sessions, lock `deploy` and `backup2` (lock, do not delete yet) |
| 2 | Escalate | Tell the security team. Root access plus a backdoor is a major incident |
| 3 | Keep evidence | Copy the logs and take a snapshot of the server |
| 4 | Investigate | Look for more backdoors: `authorized_keys`, crontab, new users, changed files. Check if this IP hit other servers |
| 5 | Clean up | Remove `backup2`, change the passwords of all users (`/etc/shadow` was read), and rebuild the server if possible |
| 6 | Prevent | `PasswordAuthentication no`, SSH only from the internal network or VPN, fail2ban, limit `deploy` to the sudo commands it needs, alert when a new user is created |

## How I would explain it

**Manager (2 lines):**
Last night an attacker guessed the password of one of our server accounts, got in, and created his own admin account. We have blocked him, changed all passwords, and added protection so repeated wrong login tries get blocked automatically.

**Technical team:**
`198.51.100.23` brute forced SSH on `web-prod-02` (14 tries on root, 9 on deploy) and logged in as `deploy` with a password at 03:21:44. Normal deploy logins use a key from `10.0.1.30`. Using deploy's sudo, it read `/etc/shadow`, created `backup2`, set its password, added it to `sudo`, and logged in as `backup2` at 03:27:39. `203.0.113.77` only tried fake usernames and failed. Treat the server as compromised: contain, keep evidence, reset all passwords, rebuild, disable password login and restrict SSH.

**Written report:**

| Field | Details |
|---|---|
| Summary | Successful SSH brute force on `web-prod-02`, followed by a backdoor admin account |
| Severity | Critical |
| Impact | Attacker had root level access. All password hashes were read |
| Timeline | 03:02 attack starts. 03:21:44 deploy login from outside. 03:22 to 03:23 shadow read, backup2 created with sudo. 03:27:39 backup2 login. 03:40 alert |
| Root cause | SSH open to the internet, password login on, guessable deploy password, full sudo for deploy |
| Fix | Block IP, lock accounts, reset all passwords, remove backdoor, rebuild server |
| Prevention | Key only SSH, SSH behind VPN, fail2ban, least privilege sudo, alert on new users |

## Interview answer

**Question:** You get a brute force alert. How do you decide if the attack was successful?

**Answer:** I check which IPs made the login attempts, and whether any of those IPs got a successful login. If a login was successful, I check what the attacker did after that.

# Case 01 — Developer Cannot SSH Into Production Server

**Data source:** Linux `/var/log/auth.log`
**Level:** Easy
**Status:** Solved

## Ticket

James (developer) cannot SSH into `web-prod-01` since the morning. Error: `Permission denied (publickey)`. He says he did not change his key. Another developer, Emma, can still log in. Internal network is `10.0.1.0/24`.

## Logs (`/var/log/auth.log`, 4 Nov)

```
Nov  4 09:52:11 web-prod-01 sshd[2211]: Accepted publickey for admin from 10.0.1.25 port 51522 ssh2: ED25519 SHA256:Xk3pQ...
Nov  4 09:52:11 web-prod-01 sshd[2211]: pam_unix(sshd:session): session opened for user admin(uid=1000) by (uid=0)
Nov  4 09:58:40 web-prod-01 sudo:    admin : TTY=pts/0 ; PWD=/home/admin ; USER=root ; COMMAND=/usr/bin/chmod -R 777 /home/james
Nov  4 09:58:40 web-prod-01 sudo: pam_unix(sudo:session): session opened for user root(uid=0) by admin(uid=1000)
Nov  4 09:58:41 web-prod-01 sudo: pam_unix(sudo:session): session closed for user root
Nov  4 10:02:17 web-prod-01 sshd[2390]: Invalid user oracle from 203.0.113.45 port 40122
Nov  4 10:02:17 web-prod-01 sshd[2390]: Connection closed by invalid user oracle 203.0.113.45 port 40122 [preauth]
Nov  4 10:02:19 web-prod-01 sshd[2392]: Invalid user test from 203.0.113.45 port 40130
Nov  4 10:02:19 web-prod-01 sshd[2392]: Connection closed by invalid user test 203.0.113.45 port 40130 [preauth]
Nov  4 10:02:21 web-prod-01 sshd[2394]: Invalid user ubuntu1 from 203.0.113.45 port 40136
Nov  4 10:02:21 web-prod-01 sshd[2394]: Connection closed by invalid user ubuntu1 203.0.113.45 port 40136 [preauth]
Nov  4 10:15:03 web-prod-01 sshd[2512]: Authentication refused: bad ownership or modes for directory /home/james
Nov  4 10:15:03 web-prod-01 sshd[2512]: Connection closed by authenticating user james 10.0.1.40 port 60218 [preauth]
Nov  4 10:15:31 web-prod-01 sshd[2519]: Authentication refused: bad ownership or modes for directory /home/james
Nov  4 10:15:31 web-prod-01 sshd[2519]: Connection closed by authenticating user james 10.0.1.40 port 60224 [preauth]
Nov  4 10:21:44 web-prod-01 sshd[2601]: Accepted publickey for emma from 10.0.1.41 port 55012 ssh2: ED25519 SHA256:Lm8vR...
Nov  4 10:21:44 web-prod-01 sshd[2601]: pam_unix(sshd:session): session opened for user emma(uid=1002) by (uid=0)
```

## Analysis

| Step | Finding |
|---|---|
| 1. Symptom | James cannot SSH into `web-prod-01`. Error: `Permission denied (publickey)` |
| 2. What I saw | 09:58:40 — `admin` ran `sudo chmod -R 777 /home/james`. 10:15:03 and 10:15:31 — James's logins from `10.0.1.40` were refused: `bad ownership or modes for directory /home/james` |
| 3. Expected | `Accepted publickey for james`, the same way Emma logged in at 10:21:44 |
| 4. Where it failed | At the key check step, before login (`[preauth]`). The network and SSH service were working |
| 5. Root cause | James's home folder was made writable by everyone (777). sshd does not trust keys in a folder that other users can change, so it refused James's key |
| 6. Next step | Fix: `chmod 755 /home/james`, `chmod 700 /home/james/.ssh`, `chmod 600 /home/james/.ssh/authorized_keys`, check `chown -R james:james /home/james`. Prevention: ask the admin why the change was made, never use 777 on home folders, use change approval on production |

### How I narrowed it down

- Emma logged in fine → SSH service, network and server are OK → the problem is only with James's account.
- I matched lines by **username, IP and PID**: PIDs 2512 and 2519 are James (`10.0.1.40`). PIDs 2390, 2392 and 2394 are a different IP (`203.0.113.45`).

## Other finding

| Item | Details |
|---|---|
| What | One outside IP `203.0.113.45` tried usernames `oracle`, `test`, `ubuntu1` between 10:02:17 and 10:02:21 |
| Meaning | Username guessing (brute force). All failed before login because the users do not exist |
| Related to James? | No |
| Severity | Medium — the attempts failed, but SSH is open to the internet |
| Action | Block the IP, allow SSH only from the internal network or VPN, add fail2ban |

Also, the `777` permission is a security risk (Medium): any user on the server could change James's `authorized_keys` and log in as him.

## How I would explain it

**Manager (2 lines):**
James could not log in to the web server because an admin changed the settings on his personal folder by mistake this morning. We will reset the settings, James will be able to log in again, and we will make sure this kind of change needs approval next time.

**Technical team:**
At 09:58:40 `admin` ran `sudo chmod -R 777 /home/james`. sshd (StrictModes) refuses keys when the home folder or `.ssh` is writable by others, so James's logins at 10:15:03 and 10:15:31 from `10.0.1.40` were refused (`bad ownership or modes for directory /home/james`). Emma logged in fine at 10:21:44, so the SSH service and network are OK. Fix: `chmod 755 /home/james`, `chmod 700 /home/james/.ssh`, `chmod 600 /home/james/.ssh/authorized_keys`, confirm `chown -R james:james /home/james`. Also ask the admin why the change was made.
Separate finding: `203.0.113.45` tried invalid usernames (`oracle`, `test`, `ubuntu1`) between 10:02:17 and 10:02:21. All failed before login. Recommend blocking the IP, limiting SSH to the internal network or a VPN, and adding fail2ban.

**Written report (short incident report):**

| Field | Details |
|---|---|
| Summary | Developer James could not SSH into `web-prod-01` |
| Impact | One user blocked from the production server. No other users affected |
| Timeline | 09:58:40 admin ran `chmod -R 777 /home/james` → 10:15:03 and 10:15:31 James's logins refused → 10:21:44 Emma logs in successfully |
| Root cause | Home folder set to 777. sshd refuses keys when the folder is writable by others |
| Fix | Restore permissions to 755 / 700 / 600 and confirm ownership |
| Prevention | No 777 on home folders, change approval for production, check with the admin |
| Other finding | Username guessing from outside IP `203.0.113.45`. Failed. Restrict SSH exposure and add fail2ban |

## Interview answer

**Question:** Emma can log in but James cannot. What does that tell you?

**Answer:** If Emma can log in, the SSH service is running, the network is fine and the server is up. So the problem is only with James's account. I filtered the logs for James and found `bad ownership or modes for directory /home/james`. Before that, the admin had run `chmod -R 777` on his home folder.


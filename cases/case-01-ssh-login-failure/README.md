# Case 01 — Developer Cannot SSH Into Production Server

**Data source:** Linux `/var/log/auth.log`
**Level:** Easy
**Status:** First attempt reviewed

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

## My first attempt (summary)

- Symptom: James cannot SSH, error `Permission denied (publickey)`.
- I found the admin command `chmod -R 777 /home/james` at 09:58:40 and the line `Authentication refused: bad ownership or modes for directory /home/james` at 10:15:03.
- I thought the attempts from `203.0.113.45` (users `oracle`, `test`, `ubuntu1`) were also James.
- My fix: change the permission to `655`.
- Extra finding: I saw the invalid user attempts, but I called them "3 users inside the internal network".

## Mistakes I made and lessons

| My answer | Correct | What I missed | Lesson |
|---|---|---|---|
| James was trying from `203.0.113.45` as `ubuntu1` | James came from `10.0.1.40` as user `james`. `203.0.113.45` is a different, outside IP | I did not match the username + IP + PID on each line | Always connect lines by **username, IP and PID** before deciding who did what |
| It failed because SSH from an external IP to an internal IP did not work | Network was fine. It failed at the **key check step**: sshd refused to trust James's key because his home folder was world-writable | `[preauth]` + "Authentication refused" means the connection reached sshd, so network is not the problem | If the log shows the server talking to the client, the network is working |
| Fix: `chmod 655` | `chmod 755 /home/james` (or 700), `chmod 700 ~/.ssh`, `chmod 600 ~/.ssh/authorized_keys`, and check `chown james:james` | On a directory, owner needs **x** to enter it. 6 = rw- (no x). Also `-R` changed every file inside, so `.ssh` must be fixed too | Know what 7/6/5/4 mean and that directories need **x** |
| No prevention step | Ask the admin why the change was made, never use 777, use change approval for production | I only fixed the problem, not the reason it happened | Every fix needs a "stop it from happening again" step |
| "3 invalid users in the internal network", Low severity | **One outside IP** guessing 3 usernames on the server (username guessing / brute force). Not related to James. Medium: it failed, but SSH is open to the internet | `203.0.113.x` is not in `10.0.1.0/24` | Check if an IP is inside or outside the network before calling it internal |
| `777` on James's home = Low | Medium. Any user on the server could change James's `authorized_keys` and log in as him | sshd blocks it for a security reason | A permission problem can also be a security risk |
| Emma works → I explained the fix | Emma works → SSH service, network and server are fine. The problem is only with **James's account** | The question was about narrowing down, not the fix | Use "who works vs who doesn't" to make the problem smaller |

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

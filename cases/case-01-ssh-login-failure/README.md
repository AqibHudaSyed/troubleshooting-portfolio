# Case 01 — Developer Cannot SSH Into Production Server

**Data source:** Linux `/var/log/auth.log`
**Level:** Easy
**Status:** Scenario given — my analysis pending

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

## My analysis

_To be added after review._

## Mistakes I made and lessons

_To be added after review._

## How I would explain it

_To be added after review._

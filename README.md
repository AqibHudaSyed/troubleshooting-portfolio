# Troubleshooting Portfolio — Aqib

Real-world style troubleshooting cases. In each case I read the logs or packet captures, find the root cause, and explain it in a way that both technical and non-technical people can understand.

## How I solve every case

| Step | Question I ask |
|---|---|
| 1. Symptom | What problem does the user or system see? |
| 2. What I saw | What do the logs show exactly (time + line)? |
| 3. Expected | What should happen when everything is normal? |
| 4. Where it failed | At which step or component did it break? |
| 5. Why (root cause) | What is the real reason? Which line proves it? |
| 6. Next step | How to fix it, and how to stop it from happening again |

Rule: every claim needs a log line as proof.

## Cases

| # | Case | Data source | Skills | Status |
|---|---|---|---|---|
| 01 | [SSH login failure](cases/case-01-ssh-login-failure/README.md) | Linux `auth.log` | sshd, sudo audit, permissions | Solved |
| 02 | [App cannot reach database](cases/case-02-app-cannot-reach-database/README.md) | tcpdump capture | DNS, TCP handshake, RST | In progress |

## Reference notes

- [tcpdump reading key](notes/tcpdump-reading-key.md)

## About the cases

All scenarios, IP addresses and hostnames are made up for practice. IPs come from documentation and private ranges.

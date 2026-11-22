# Troubleshooting Portfolio — Aqib

Real-world style troubleshooting cases from general IT work: Linux, networking, DNS, web servers, cloud, Windows and security. In each case I read the logs or packet captures, find the root cause, and explain it so both technical and non-technical people can understand.

## How I read any log

| # | Step | What I do |
|---|---|---|
| 1 | Set the scope | What is the problem, when did it start, which server or user |
| 2 | Know the line format | Where is the time, the source and the message |
| 3 | Remove noise | Keep only lines for my user, IP and time. Look for words like `error`, `fail`, `denied`, `refused`, `timeout` |
| 4 | Group lines | Connect lines by PID, IP, user or request ID |
| 5 | Build a timeline | Find the **change** first, then the effect |
| 6 | Compare good vs bad | What works and what fails — the difference is the clue |
| 7 | Conclude with proof | Every claim has a log line |

## How I solve every case

| Step | Question I ask |
|---|---|
| 1. Symptom | What problem does the user or system see? |
| 2. What I saw | What do the logs show exactly (time + line)? |
| 3. Expected | What should happen when everything is normal? |
| 4. Where it failed | At which step or component did it break? |
| 5. Why (root cause) | What is the real reason? Which line proves it? |
| 6. Next step | How to fix it, and how to stop it from happening again |

## Topics and cases

| # | Topic | Reading key | Cases |
|---|---|---|---|
| 01 | Linux system and auth logs | [Key](01-linux-system-and-auth-logs/reading-key.md) | [Case 01 — SSH login failure](01-linux-system-and-auth-logs/case-01-ssh-login-failure/README.md) ✅ |
| 02 | Network and tcpdump | [Key](02-network-and-tcpdump/reading-key.md) | [Case 02 — App cannot reach database](02-network-and-tcpdump/case-02-app-cannot-reach-database/README.md) ⏳ |
| 03 | DNS | — | Coming soon |
| 04 | Web servers (Nginx / Apache) | — | Coming soon |
| 05 | Disk, CPU, memory, processes | — | Coming soon |
| 06 | Services and app / database logs | — | Coming soon |
| 07 | Firewall and SSL/TLS | — | Coming soon |
| 08 | Windows Event Logs | — | Coming soon |
| 09 | Cloud (AWS) | — | Coming soon |
| 10 | Security alerts (SIEM) | — | Coming soon |
| 11 | Multi-source incidents | — | Coming soon |

## About the cases

All scenarios, names, IP addresses and hostnames are made up for practice. IPs come from documentation and private ranges.

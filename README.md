# Troubleshooting Portfolio by Aqib

Real-world style troubleshooting cases from general IT work: Linux, networking, DNS, web servers, cloud, Windows and security. In each case I read the logs or packet captures, find the root cause, and explain it so both technical and non-technical people can understand.

## How I troubleshoot

I use the same 5 questions for every problem.

| # | Question | What I look for |
|---|---|---|
| 1 | What broke? | The symptom: what, who, when, where |
| 2 | What changed? | The change, or the first bad line in the log |
| 3 | What is different? | What works vs what fails. The difference is the clue |
| 4 | What is the proof? | One log line for every claim. No guesses |
| 5 | What next? | Fix it now, and stop it from happening again. For a security incident, contain it first |

## Topics and cases

| # | Topic | Reading key | Cases |
|---|---|---|---|
| 01 | Linux system and auth logs | [Key](01-linux-system-and-auth-logs/reading-key.md) | [Case 01: SSH login failure](01-linux-system-and-auth-logs/case-01-ssh-login-failure/README.md) ✅<br>[Case 02: Brute force and a new user](01-linux-system-and-auth-logs/case-02-brute-force-and-suspicious-login/README.md) ✅ |
| 02 | Network and tcpdump | [Key](02-network-and-tcpdump/reading-key.md) | [Case 01: App cannot reach database](02-network-and-tcpdump/case-01-app-cannot-reach-database/README.md) ✅<br>[Case 02: API timeout, no reply](02-network-and-tcpdump/case-02-api-timeout-no-reply/README.md) ✅ |
| 03 | DNS | Coming soon | Coming soon |
| 04 | Web servers (Nginx / Apache) | Coming soon | Coming soon |
| 05 | Disk, CPU, memory, processes | Coming soon | Coming soon |
| 06 | Services and app / database logs | Coming soon | Coming soon |
| 07 | Firewall and SSL/TLS | Coming soon | Coming soon |
| 08 | Windows Event Logs | Coming soon | Coming soon |
| 09 | Cloud (AWS) | Coming soon | Coming soon |
| 10 | Security alerts (SIEM) | Coming soon | Coming soon |
| 11 | Multi-source incidents | Coming soon | Coming soon |

## Progress

See [coverage.md](coverage.md) for what I have practiced so far.

## About the cases

All scenarios, names, IP addresses and hostnames are made up for practice. IPs come from documentation and private ranges.

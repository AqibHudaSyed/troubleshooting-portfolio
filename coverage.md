# Coverage Tracker

What I have practiced so far. Updated after every case.

✅ done · 🟡 started · ⬜ not yet

## Topics

Goal: 50 cases, about 1 hour a day.

- **Phase 1** (until about 20 Dec): 21 cases, so every topic is covered at least once.
- **Phase 2** (after that): the remaining cases, marked "Phase 2" below, to go deeper into each topic.

| # | Topic | Case 1 | Case 2 | Case 3 | Case 4 |
|---|---|---|---|---|---|
| 01 | Linux system and auth logs | ✅ | ✅ | ⬜ | Phase 2 |
| 02 | Network and tcpdump | ✅ | ✅ | ⬜ | Phase 2 |
| 03 | DNS | ⬜ | ⬜ | Phase 2 | Phase 2 |
| 04 | Web servers and HTTP | ⬜ | ⬜ | Phase 2 | Phase 2 |
| 05 | Disk, CPU, memory, processes | ⬜ | ⬜ | Phase 2 | Phase 2 |
| 06 | Services, cron, app and DB logs | ⬜ | ⬜ | Phase 2 | Phase 2 |
| 07 | Firewall and SSL/TLS | ⬜ | ⬜ | Phase 2 | Phase 2 |
| 08 | Windows and Active Directory | ⬜ | Phase 2 | Phase 2 | Phase 2 |
| 09 | IT support basics (DHCP, VPN, email) | ⬜ | Phase 2 | Phase 2 | Phase 2 |
| 10 | AWS cloud | ⬜ | Phase 2 | Phase 2 | Phase 2 |
| 11 | Security alerts (SIEM) | ⬜ | Phase 2 | Phase 2 | Phase 2 |
| 12 | Multi-source incidents | ⬜ | Phase 2 | Phase 2 | Phase 2 |

## Logs

| Log | Status | Practiced in |
|---|---|---|
| `auth.log` / `secure` | ✅ | Linux 1, Linux 2, grep drills 1 and 2 |
| tcpdump output | ✅ | Network 1, Network 2, flags drill |
| `syslog` / `messages` / `journalctl` | ⬜ | |
| Nginx `access.log` / `error.log` | ⬜ | |
| `kern.log` / `dmesg` | ⬜ | |
| Firewall (ufw, iptables) | ⬜ | |
| App and database logs | ⬜ | |
| Cron, `dpkg` / `dnf` | ⬜ | |
| DNS tools (`dig`, `nslookup`) | ⬜ | |
| Windows Event Logs | ⬜ | |
| AWS (CloudTrail, VPC Flow Logs) | ⬜ | |
| SIEM alerts | ⬜ | |

## Commands

| Command | Status |
|---|---|
| `grep`, `grep -c`, `grep -v`, `grep -i` | ✅ |
| `grep -A` / `-B`, time window (`03:2[0-9]`) | ✅ |
| `head -1`, `tail -1` | ✅ |
| `wc -l` | ✅ |
| `awk '{print $N}'`, `$(NF-N)` | ✅ |
| `sort \| uniq -c \| sort -rn` | ✅ |
| `ss -tlnp`, `curl` | 🟡 read only |
| `tail -f`, `less` | ⬜ |
| `journalctl` | ⬜ |
| `dig`, `nslookup` | ⬜ |
| `df`, `du`, `free`, `top` | ⬜ |
| `systemctl status` | 🟡 read only |
| tcpdump capture to file (`-w`, `-r`, `-C`, `-W`, filters) | 🟡 read only |
| `nohup`, `&`, `jobs`, `kill` (background jobs) | ⬜ |
| `scp`, `rsync` (copy files between servers) | ⬜ |

## Other practice

| Item | Status |
|---|---|
| Interview rounds | ⬜ 0 done |
| Playbooks | ⬜ 0 done |
| Real work story (STAR) | ⬜ |

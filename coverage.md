# Coverage Tracker

What I have practiced so far. Updated after every case.

✅ done · 🟡 started · ⬜ not yet

## Topics

Core plan (about 1 hour a day, until 20 Dec): 21 cases. "later" = extra cases after that.

| # | Topic | Case 1 | Case 2 | Case 3 | Case 4 |
|---|---|---|---|---|---|
| 01 | Linux system and auth logs | ✅ | ✅ | ⬜ | later |
| 02 | Network and tcpdump | ✅ | ✅ | ⬜ | later |
| 03 | DNS | ⬜ | ⬜ | later | later |
| 04 | Web servers and HTTP | ⬜ | ⬜ | later | later |
| 05 | Disk, CPU, memory, processes | ⬜ | ⬜ | later | later |
| 06 | Services, cron, app and DB logs | ⬜ | ⬜ | later | later |
| 07 | Firewall and SSL/TLS | ⬜ | ⬜ | later | later |
| 08 | Windows and Active Directory | ⬜ | later | later | later |
| 09 | IT support basics (DHCP, VPN, email) | ⬜ | later | later | later |
| 10 | AWS cloud | ⬜ | later | later | later |
| 11 | Security alerts (SIEM) | ⬜ | later | later | later |
| 12 | Multi-source incidents | ⬜ | later | later | later |

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

# Case 01: Web App Cannot Reach the Database

**Data source:** tcpdump capture on the app server
**Level:** Easy to Medium
**Status:** Solved

## Ticket

The company website shows "500 Internal Server Error" since about 9:00 AM. The app log on `app-01` (10.0.2.10) says: `could not connect to database db.internal:5432`.

Extra info:
- The database is PostgreSQL on `db-01` (10.0.2.20), port 5432.
- Both servers are in the same AWS VPC. The VPC DNS server is 10.0.0.2.
- Change log: last night at 11 PM the DB team did OS patching on `db-01` and rebooted it.
- Nothing was changed on `app-01`.

## Capture (taken on app-01)

```
$ sudo tcpdump -i eth0 -nn "host 10.0.2.20 or port 53"
10:41:02.100211 IP 10.0.2.10.48211 > 10.0.0.2.53: 4521+ A? db.internal. (29)
10:41:02.101034 IP 10.0.0.2.53 > 10.0.2.10.48211: 4521 1/0/0 A 10.0.2.20 (45)
10:41:02.101512 IP 10.0.2.10.51344 > 10.0.2.20.5432: Flags [S], seq 2873401123, win 64240, options [mss 1460,sackOK,TS val 401223 ecr 0,nop,wscale 7], length 0
10:41:02.101903 IP 10.0.2.20.5432 > 10.0.2.10.51344: Flags [R.], seq 0, ack 2873401124, win 0, length 0
10:41:03.104417 IP 10.0.2.10.51346 > 10.0.2.20.5432: Flags [S], seq 1190023877, win 64240, options [mss 1460,sackOK,TS val 402226 ecr 0,nop,wscale 7], length 0
10:41:03.104788 IP 10.0.2.20.5432 > 10.0.2.10.51346: Flags [R.], seq 0, ack 1190023878, win 0, length 0
10:41:05.222019 IP 10.0.2.10.40122 > 10.0.2.20.22: Flags [S], seq 3301298410, win 64240, options [mss 1460,sackOK,TS val 404344 ecr 0,nop,wscale 7], length 0
10:41:05.222540 IP 10.0.2.20.22 > 10.0.2.10.40122: Flags [S.], seq 776120933, ack 3301298411, win 65160, options [mss 1460,sackOK,TS val 9912034 ecr 404344,nop,wscale 7], length 0
10:41:05.222601 IP 10.0.2.10.40122 > 10.0.2.20.22: Flags [.], ack 1, win 502, length 0
10:41:05.230114 IP 10.0.2.20.22 > 10.0.2.10.40122: Flags [P.], seq 1:42, ack 1, win 509, length 41
^C
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```

## Analysis

| Step | Finding |
|---|---|
| 1. Symptom | The website shows "500 Internal Server Error" since about 9 AM. The app log says `could not connect to database db.internal:5432` |
| 2. What I saw | At 10:41:02.101512 and 10:41:03.104417 app-01 sent SYN to db-01 on port 5432. Each time db-01 replied straight away with `[R.]` (RST) at 10:41:02.101903 and 10:41:03.104788 |
| 3. Expected | A normal TCP handshake on port 5432: `[S]` → `[S.]` → `[.]`, then data |
| 4. Where it failed | On db-01, port 5432, at the very first step of the TCP connection |
| 5. Root cause | PostgreSQL was not listening on port 5432 after last night's patching and reboot. The reboot was the trigger. The service most likely did not start again |
| 6. Next step | Fix: start PostgreSQL and enable it at boot. Prevention: after every patch or reboot, run a short check list (service status, port listening, app health) before closing the change |

### How I narrowed it down

| Check | Proof | Result |
|---|---|---|
| Is DNS working? | 10:41:02.101034 `4521 1/0/0 A 10.0.2.20` | Yes. `db.internal` gives the right IP |
| Is db-01 up and the network fine? | Port 22: `[S]` → `[S.]` → `[.]` → `[P.]` at 10:41:05 | Yes. Full handshake and SSH banner |
| What happens on port 5432? | `[S]` → `[R.]` within half a millisecond | db-01 refuses. Nothing is listening on 5432 |
| What changed? | Change log: patching and reboot of db-01 at 11 PM | The problem started after this |

### RST vs no reply

| In the capture | Meaning | What I check |
|---|---|---|
| `[S]` → `[S.]` | Port is open, service is running | The problem is higher up (app, password, query) |
| `[S]` → `[R.]` straight away | Host is up, network is fine, nothing is listening on the port | Is the service running? Is it listening on the right IP and port? |
| `[S]` repeated, no reply | Packet is dropped on the way | Firewall, Security Group, NACL, routing, or host down |

### Commands to confirm on db-01

```
sudo ss -tlnp | grep 5432
sudo systemctl status postgresql
sudo journalctl -u postgresql --since "yesterday 23:00"
```

Fix:

```
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

## How I would explain it

**Manager (2 lines):**
The website is down because the database did not come back after last night's maintenance. The DB team is starting it now, and from now on we will check that everything is running again after every maintenance.

**Technical team:**
DNS is fine: `db.internal` resolves to 10.0.2.20. The TCP handshake on port 22 works, so the host and network are fine. Every SYN to port 5432 gets an instant RST, so nothing is listening on 5432. This started after the 11 PM patch reboot. Check with `ss -tlnp | grep 5432` and `systemctl status postgresql`. Fix: start the service and enable it at boot.

**Written report:**

| Field | Details |
|---|---|
| Summary | Website returned 500 errors because the app could not connect to the database |
| Impact | Website down for all users from about 9 AM |
| Timeline | 11 PM: db-01 patched and rebooted. About 9 AM: 500 errors reported. 10:41: capture shows RST from db-01 on port 5432 |
| Root cause | PostgreSQL was not listening on port 5432 after the reboot |
| Fix | Start PostgreSQL and enable it at boot |
| Prevention | Post-change check list: service running, port listening, app health check. Add monitoring on port 5432 |

## Interview answer

**Question:** What is the difference between getting an RST and getting no reply, and what do you check in each case?

**Answer:** An RST means my packet reached the server and the server said no, because nothing is listening on that port. So I check on the server if the service is running and which port it is listening on. No reply means the packet was dropped on the way. So I check firewalls, Security Groups, routing, and if the server is up.

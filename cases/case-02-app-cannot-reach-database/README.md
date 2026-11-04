# Case 02 — Web App Cannot Reach the Database

**Data source:** tcpdump capture on the app server
**Level:** Easy–Medium
**Status:** Scenario given — my analysis pending

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

## My analysis

_To be added after review._

## Mistakes I made and lessons

_To be added after review._

## How I would explain it

_To be added after review._

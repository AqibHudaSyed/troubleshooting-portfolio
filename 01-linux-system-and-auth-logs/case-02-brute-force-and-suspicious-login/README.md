# Case 02: Brute Force Alert and a User Nobody Created

**Data source:** Linux `/var/log/auth.log` ([full log](auth.log), 89 lines)
**Level:** Medium (guided)
**Status:** Scenario given, analysis pending

## Ticket

At 03:40 the monitoring tool raised an alert: "High number of failed SSH logins on `web-prod-02`". In the morning, Olivia (sysadmin) also saw a user called `backup2` on the server. She did not create it.

| Item | Details |
|---|---|
| Server | `web-prod-02`, a web server reachable from the internet on port 22 |
| Internal network | `10.0.1.0/24` |
| Known users | `olivia` (sysadmin, logs in from 10.0.1.25), `deploy` (used by the CI server 10.0.1.30 to deploy code, has sudo rights) |
| SSH setting | Password login is still enabled on this old server |

## Commands used

```
# How many failed logins in total
grep -c "Failed password" auth.log

# Failed logins per IP (IP is the 4th field from the end)
grep "Failed password" auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

# Failed logins per real user (lines without "invalid user")
grep "Failed password" auth.log | grep -v "invalid user" | awk '{print $9}' | sort | uniq -c | sort -rn
```

Output:

```
34

     29 198.51.100.23
      5 203.0.113.77

     14 root
      9 deploy
```

## Analysis

_To be added after review._

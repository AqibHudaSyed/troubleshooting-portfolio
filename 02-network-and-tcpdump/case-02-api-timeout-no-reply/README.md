# Case 02: API Calls Time Out, SYN Gets No Reply

**Data source:** tcpdump on the client and on the server, `ss` and `curl` on the server
**Level:** Medium
**Status:** Scenario given, analysis pending

## Ticket

Daniel (developer): calls from `app-02` to the API on `api-01` (port 8080) time out after 20 to 25 seconds. It worked yesterday. SSH to the server still works.

| Item | Details |
|---|---|
| `app-02` | 10.0.3.15 (client) |
| `api-01` | 10.0.3.40 (Java API on port 8080) |
| Network | Both are in the same AWS VPC |
| Change log | Yesterday evening the cloud team removed the old Security Group from `api-01` and attached a new one, `api-sg-v2`, as part of a cleanup |
| `api-01` | No code or config change |

## Capture 1 (on app-02)

```
$ sudo tcpdump -i eth0 -nn host 10.0.3.40
10:20:01.000112 IP 10.0.3.15.45210 > 10.0.3.40.8080: Flags [S], seq 3011201, win 64240, options [mss 1460,sackOK,TS val 811201 ecr 0,nop,wscale 7], length 0
10:20:02.010245 IP 10.0.3.15.45210 > 10.0.3.40.8080: Flags [S], seq 3011201, win 64240, options [mss 1460,sackOK,TS val 812211 ecr 0,nop,wscale 7], length 0
10:20:04.030511 IP 10.0.3.15.45210 > 10.0.3.40.8080: Flags [S], seq 3011201, win 64240, options [mss 1460,sackOK,TS val 814231 ecr 0,nop,wscale 7], length 0
10:20:08.070833 IP 10.0.3.15.45210 > 10.0.3.40.8080: Flags [S], seq 3011201, win 64240, options [mss 1460,sackOK,TS val 818271 ecr 0,nop,wscale 7], length 0
10:20:16.150901 IP 10.0.3.15.45210 > 10.0.3.40.8080: Flags [S], seq 3011201, win 64240, options [mss 1460,sackOK,TS val 826351 ecr 0,nop,wscale 7], length 0
10:20:30.400112 IP 10.0.3.15.51002 > 10.0.3.40.22: Flags [S], seq 7700120, win 64240, options [mss 1460,sackOK,TS val 840600 ecr 0,nop,wscale 7], length 0
10:20:30.400690 IP 10.0.3.40.22 > 10.0.3.15.51002: Flags [S.], seq 9120034, ack 7700121, win 65160, options [mss 1460,sackOK,TS val 551201 ecr 840600,nop,wscale 7], length 0
10:20:30.400733 IP 10.0.3.15.51002 > 10.0.3.40.22: Flags [.], ack 1, win 502, length 0
^C
8 packets captured
8 packets received by filter
0 packets dropped by kernel
```

## Capture 2 (on api-01, at the same time, API call retried)

```
$ sudo tcpdump -i eth0 -nn port 8080
^C
0 packets captured
0 packets received by filter
0 packets dropped by kernel
```

## Checks on api-01

```
$ sudo ss -tlnp | grep 8080
LISTEN 0  4096  0.0.0.0:8080  0.0.0.0:*  users:(("java",pid=1432,fd=45))

$ curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080/health
200
```

## Analysis

_To be added after review._

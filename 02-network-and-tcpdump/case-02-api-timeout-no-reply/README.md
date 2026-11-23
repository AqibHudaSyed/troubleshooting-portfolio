# Case 02: API Calls Time Out, SYN Gets No Reply

**Data source:** tcpdump on the client and on the server, `ss` and `curl` on the server
**Level:** Medium
**Status:** Solved

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

| Step | Finding |
|---|---|
| 1. Symptom | API calls from `app-02` to `api-01` on port 8080 time out. SSH to `api-01` still works |
| 2. What I saw | From 10:20:01 to 10:20:16 app-02 sent the same SYN (`seq 3011201`) five times to 10.0.3.40:8080. No reply came back. On api-01, tcpdump on port 8080 captured 0 packets |
| 3. Expected | On port 8080, the same as port 22: `[S]` → `[S.]` → `[.]` within milliseconds |
| 4. Where it failed | Between app-02 and api-01. The SYN never reached api-01, so it was dropped outside the server, at the first step of the TCP connection |
| 5. Root cause | Trigger: yesterday the old Security Group on api-01 was replaced with `api-sg-v2`. Real cause: `api-sg-v2` allows port 22 but has no inbound rule for TCP 8080 |
| 6. Next step | Fix: add an inbound rule to `api-sg-v2` for TCP 8080 from `10.0.3.15/32` (or the app servers' subnet or Security Group). Test from app-02 with `curl http://10.0.3.40:8080/health`. Prevention: compare old and new Security Group rules before a swap, and test all app ports after the change |

### How I narrowed it down

| Check | Proof | Result |
|---|---|---|
| Is the API app running? | `ss`: `java` LISTEN on `0.0.0.0:8080`. `curl localhost:8080/health` → `200` | Yes. The app is fine |
| Is the server reachable? | Port 22: `[S]` → `[S.]` → `[.]` at 10:20:30 | Yes. Host and network path work for SSH |
| What happens on 8080? | Same SYN sent again after 1, 2, 4, 8 seconds, no reply | Packet is dropped, not refused |
| Does the SYN reach api-01? | tcpdump on api-01 port 8080: 0 packets | No. It is blocked before the server |
| What changed? | Security Group swap yesterday evening | Problem started after this |

### Where the packet stopped

| On the server's tcpdump | Meaning |
|---|---|
| SYN seen, RST sent back | Packet arrived, nothing listening on the port |
| SYN seen, no reply sent | Packet arrived, the server's own firewall (iptables/ufw) dropped it. tcpdump sees packets before the host firewall |
| Nothing seen | Packet never arrived. Blocked outside: Security Group, NACL or routing |

### Retransmission

The client sends the same SYN again (same `seq`) when no reply comes, and waits longer each time: 1s, 2s, 4s, 8s. Same seq + growing gap = no answer from the other side.

## How I would explain it

**Manager (2 lines):**
A network setting on the API server was removed during a cleanup change yesterday evening, so the app can no longer talk to the API. Once the setting is added back the service will work again, and future changes like this will be tested before they are closed.

**Technical team:**
app-02 sends SYN to api-01:8080 and retransmits it after 1, 2, 4 and 8 seconds with no reply. tcpdump on api-01 shows no packets on 8080, so the SYN never reaches the server. The app is fine: java listens on 0.0.0.0:8080 and the local health check returns 200. Port 22 works. This started after yesterday's Security Group swap. `api-sg-v2` has no inbound rule for TCP 8080. Fix: allow TCP 8080 from `10.0.3.15/32` (or the app subnet), then test with curl from app-02.

**Written report:**

| Field | Details |
|---|---|
| Summary | API calls from app-02 to api-01 timed out |
| Impact | Features that use the API were down. SSH was not affected |
| Timeline | Yesterday evening: Security Group swapped to `api-sg-v2`. 10:20: capture shows SYN retries with no reply and nothing arriving on api-01 |
| Root cause | `api-sg-v2` has no inbound rule for TCP 8080 |
| Fix | Add inbound TCP 8080 from the app servers |
| Prevention | Compare rules before swapping Security Groups. Test every app port after network changes |

## Interview answer

**Question:** The client shows the SYN going out, but the server's tcpdump shows nothing. What does that mean?

**Answer:** It means the packet was dropped outside the server and never reached it. On the server, `ss` shows the Java app is listening on port 8080, so there is no problem on the server side. If the port was closed I would see an RST. So the problem is in the firewall or the Security Group inbound rule on the way.

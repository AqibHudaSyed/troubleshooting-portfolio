# tcpdump Reading Key

tcpdump is a command-line tool that captures network packets going in and out of a machine. It is useful on servers where there is no GUI. You can save the capture as a `.pcap` file and open it later in Wireshark.

## 1. Most used options

| Option | What it does |
|---|---|
| `-D` | List the network interfaces you can capture on |
| `-i eth0` / `-i any` | Capture on one interface / on all interfaces |
| `-c 20` | Stop after 20 packets |
| `-n` | Do not turn IPs into hostnames |
| `-nn` | Do not turn IPs or ports into names (shows `443`, not `https`) — best for troubleshooting |
| `-w file.pcap` | Save packets to a file (nothing is printed on screen) |
| `-r file.pcap` | Read a saved file (no sudo needed) |
| `-v`, `-vv`, `-vvv` | Show more detail (IP header: TTL, ID, flags, total length) |
| `-A` | Show packet content as text (useful for plain HTTP) |
| `-X` | Show packet content in hex + text |
| `-e` | Show Ethernet (MAC) header |
| `-S` | Show full (absolute) sequence numbers instead of relative ones |
| `-s0` | Capture the full packet (use it if output ends with `[|proto]`) |

Capturing needs `sudo`. Press `Ctrl+C` to stop.

## 2. Filters

| Filter | Example | Meaning |
|---|---|---|
| `host` | `host 10.0.2.20` | Traffic to or from this IP |
| `src` / `dst` | `src 10.0.2.10` | Only traffic from / to this IP |
| `net` | `net 10.0.2.0/24` | Traffic of a whole subnet |
| `port` | `port 443` | Traffic on this port (either side) |
| `portrange` | `portrange 8000-8100` | A range of ports |
| Protocol | `tcp`, `udp`, `icmp`, `arp` | Only this protocol |
| `and`, `or`, `not` | `host 10.0.2.20 and not port 22` | Combine filters |

Use quotes when you use brackets: `"port 5432 and (src 10.0.2.10 or src 10.0.2.20)"`

Good habit: `sudo tcpdump -i eth0 -nn -w case.pcap host <ip>` — capture to a file first, read later.

## 3. Reading a TCP line

```
10:41:02.101512 IP 10.0.2.10.51344 > 10.0.2.20.5432: Flags [S], seq 2873401123, win 64240, options [mss 1460,sackOK,TS val 401223 ecr 0,nop,wscale 7], length 0
```

| Part | Value | Meaning |
|---|---|---|
| Time | `10:41:02.101512` | When the packet was seen (local clock, microseconds) |
| `IP` | `IP` | IPv4 (`IP6` = IPv6) |
| Source | `10.0.2.10.51344` | IP + port. The **last number after the dot is the port** |
| `>` | | Direction of the packet |
| Destination | `10.0.2.20.5432` | IP + port (5432 = PostgreSQL) |
| `Flags [S]` | | TCP flags (see table below) |
| `seq` | `2873401123` | Sequence number — which bytes this packet carries |
| `ack` | | Next byte I expect from the other side |
| `win` | `64240` | How much data the sender can receive right now (buffer space) |
| `options` | `mss, sackOK, TS, wscale` | Extra TCP settings, mostly seen in SYN packets |
| `length` | `0` | Size of the **data (payload)** only, not headers |

### TCP flags

| Flag | Name | Meaning in simple words |
|---|---|---|
| `[S]` | SYN | "I want to connect" |
| `[S.]` | SYN-ACK | "OK, I accept" |
| `[.]` | ACK | "Got it" |
| `[P.]` | PUSH + ACK | "Here is data" |
| `[F.]` | FIN + ACK | "I am done, closing nicely" |
| `[R]` / `[R.]` | RST | "Stop — reset this connection" |

The dot `.` always means ACK.

### seq and ack in simple words

- The first packet shows big absolute numbers. After that tcpdump shows **relative** numbers (starting from 1) so they are easy to read.
- `seq 1:113` = this packet carries bytes 1 to 112 → `length 112`.
- `ack 113` from the other side = "I received everything up to 112, send byte 113 next."

### SYN options

| Option | Meaning |
|---|---|
| `mss 1460` | Biggest data chunk this side can receive in one packet |
| `sackOK` | Selective ACK allowed (resend only the missing pieces) |
| `TS val / ecr` | Timestamps, used to measure round-trip time |
| `wscale 7` | Window scaling — lets the window grow beyond 64 KB |
| `nop` | Padding, ignore it |

## 4. Extra fields with `-v`

```
IP (tos 0x0, ttl 64, id 9792, offset 0, flags [DF], proto TCP (6), length 88)
```

| Field | Meaning |
|---|---|
| `ttl 64` | Hops left before the packet is dropped. Linux starts at 64, Windows at 128 — a hint about the OS |
| `id` | Used to put fragments back together |
| `flags [DF]` | Don't Fragment |
| `proto TCP (6)` | Layer 4 protocol (TCP 6, UDP 17, ICMP 1) |
| `length 88` | **Whole IP packet** including headers (different from the TCP `length` at the end) |

## 5. Other common lines

| Protocol | Example | How to read it |
|---|---|---|
| DNS query | `10.0.2.10.48211 > 10.0.0.2.53: 4521+ A? db.internal. (29)` | Query ID 4521, `+` = recursion wanted, `A?` = "give me the IPv4 of db.internal" |
| DNS reply | `10.0.0.2.53 > 10.0.2.10.48211: 4521 1/0/0 A 10.0.2.20 (45)` | Same ID 4521. `1/0/0` = 1 answer / 0 authority / 0 additional. Answer is 10.0.2.20 |
| DNS error | `... 4521 NXDomain 0/1/0` | Name does not exist. `ServFail` = DNS server failed |
| ICMP | `ICMP echo request` / `echo reply` | ping going / ping coming back |
| ICMP error | `ICMP ... port 53 unreachable` | Nothing listening on that UDP port |
| ARP | `ARP, Request who-has 10.0.2.1 tell 10.0.2.10` | "Who has this IP? Tell me your MAC" |

## 6. Footer after Ctrl+C

| Line | Meaning |
|---|---|
| `packets captured` | Packets that matched the filter and were shown/saved |
| `received by filter` | Packets the filter checked |
| `dropped by kernel` | Packets lost because tcpdump was too slow — if high, your capture is incomplete |

## 7. Ports

| Range | Type |
|---|---|
| 0–1023 | Well-known (22 SSH, 53 DNS, 80 HTTP, 443 HTTPS) |
| 1024–49151 | Registered (3306 MySQL, 5432 PostgreSQL, 5060 SIP) |
| 49152–65535 | Ephemeral — random client-side ports |

## 8. Normal vs problem patterns

| What you see | What it usually means |
|---|---|
| `[S]` → `[S.]` → `[.]` → `[P.]` data → `[F.]` | Healthy connection |
| `[S]` → `[R.]` back from server | Server is up but **nothing is listening on that port** (or it rejects actively) |
| `[S]` repeated, no reply at all | Packet is being **dropped** — firewall/security group, routing, or host down |
| `[S]` → `[S.]` then later `[R]` | Connection made, then reset by the app or a middle device |
| DNS query with no reply | DNS server not reachable / UDP 53 blocked |
| DNS reply `NXDomain` | Wrong name or missing record |
| `ICMP host unreachable` | A router says it cannot reach the host |

## Sources I learned from

- Opensource.com — An introduction to using tcpdump at the Linux command line
- Comparitech — tcpdump cheat sheet
- Packet Pushers — Masterclass: tcpdump, interpreting output

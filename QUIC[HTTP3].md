## Advantages of QUIC/HTTP/3 over HTTP/2

| Feature | HTTP/2 (TCP) | HTTP/3 (QUIC) |
|---|---|---|
|**Multiplexing**	| Application-layer — multiple streams share one TCP byte stream; a single lost packet blocks all streams (transport-level HOL)	|Transport-layer — each stream has independent sequence numbers & retransmission; one lost packet only blocks its own stream|
| **Connection setup** | TCP 3-way handshake + TLS handshake = 2–3 RTTs | Combined transport + crypto = **1 RTT** (or **0-RTT** for resumption) |
| **0-RTT data** | Not possible | Client can send request data in the **very first packet** on reconnect |
| **Connection migration** | Not supported — IP/port change kills the connection | Survives Wi-Fi ↔ cellular switch seamlessly via **Connection ID** |
| **TLS** | Optional (TLS 1.2 or 1.3) | **Mandatory TLS 1.3**, built into QUIC |
| **Forward Error Correction (FEC)** | None | QUIC can reconstruct lost packets **without retransmission** |
| **Congestion control** | Kernel TCP (limited, slow to adapt) | **User-space**, per-connection, more flexible algorithms (e.g., BBR) |
| **Header compression** | HPACK (vulnerable to cache-poisoning attacks) | **QPACK** (stateless, no cache-poisoning risk) |
| **Performance on lossy networks** | Degrades ~50%+ at 15% packet loss | **55% better** under same conditions |
| **Protocol versioning** | Breaking changes require new protocol | QUIC can **evolve in user-space** without kernel updates |

---

## Why HTTP/3 Is Still Not Dominant

| Barrier | Explanation |
|---|---|
| **UDP blocked by firewalls** | Many corporate/ISP firewalls and legacy middleboxes block or deprioritize UDP. Browsers silently fall back to HTTP/2. |
| **Slower on fast networks (>500 Mbps)** | A 2024 ACM paper ("QUIC is not Quick Enough over Fast Internet") showed QUIC delivers **up to 45% less throughput** than TCP/HTTP/2 at 1 Gbps. As fiber broadband expands, more users cross this threshold where HTTP/3 actually *hurts*. |
| **Higher CPU cost** | Encryption and stream management happen in **user-space** (not kernel), increasing server CPU load significantly at scale. |
| **Poor library/server support** | `curl` still marks HTTP/3 as experimental, Apache has no support, OpenSSL's QUIC integration is incompatible with most existing implementations. Most language standard libraries lack first-class support. |
| **Discovery problem** | Browsers need `Alt-Svc` headers or DNS HTTPS records to *learn* a server supports HTTP/3. **First visits** almost never use it, inflating the "supported but unused" gap (38.8% advertise vs ~21% actually used). |
| **Requires multi-layer coordination** | Unlike a TLS upgrade (just bump the library), HTTP/3 needs simultaneous changes at **transport (UDP), crypto (QUIC), and application (QPACK)** layers — a much heavier operational lift. |
| **Security tooling lag** | WAFs, IDS/IPS, and DDoS protection tools were designed for TCP. Inspecting encrypted UDP/QUIC traffic is harder, making security teams cautious. |
| **Marginal gains for most users** | On stable desktop networks the improvement is only ~5%. For most dev teams, the effort-to-benefit ratio doesn't justify the migration. |

**Bottom line:** HTTP/3's biggest wins are on **lossy, mobile, or high-latency networks** — exactly where adoption is growing fastest (India, Brazil, Mexico lead at 29–30% vs 21% global). But on the fast, clean networks of developed markets, the protocol can actually be *slower*, which is the main structural reason adoption plateaued around 2025–2026.


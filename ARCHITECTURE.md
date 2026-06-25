# London + Mumbai Self-Hosted VPN (Pi 5)

Status: BUILDING (decisions locked)
Owner: Vincent Pereira
Hardware: Raspberry Pi 5 (London flat) + an old always-on laptop (Mumbai home)

------------------------------------------------------------------------------
## 1. Goals
------------------------------------------------------------------------------
1. Remote access to the London home network from anywhere + a UK/London IP when travelling.
2. Watch Indian OTT from London (JioHotstar, Airtel Xstream) and use India-only
   sites (HP Gas, Adani Electricity) already paid for in Mumbai.
3. Network-wide ad/tracker blocking + DNS privacy.
4. ISP-traffic anonymisation (secondary; partial only - see caveat section 8).
5. Free / near-zero ongoing cost. Prefer GBP 0.

Devices: 4 mobiles (Android + iOS), 3 Windows laptops, 1 TV, 1 Pi 5.
Users: 4 simultaneous (family).

------------------------------------------------------------------------------
## 2. Decided architecture: two-node Tailscale mesh + Pi-hole/Unbound
------------------------------------------------------------------------------
Two research findings locked the design:

A) Community Fibre (London) uses CGNAT by default (public IP is a GBP 4/mo add-on
   we do NOT need). Classic port-forward WireGuard cannot work here.
   -> Use Tailscale (free WireGuard mesh), which punches through CGNAT. No public
      IP or port forwarding required.

B) Indian OTT blocks all datacenter/VPS/commercial-VPN IPs. Only a RESIDENTIAL
   Indian IP works reliably. A cheap India VPS will NOT unblock OTT.
   -> Put the India exit node on the Mumbai home's own broadband (residential IP).

Architecture:

```mermaid
flowchart TB
  subgraph LDN["London flat"]
    CF["Community Fibre broadband<br/>(CGNAT - no public IPv4)"]
    Pi["Raspberry Pi 5 - HUB<br/>Tailscale: UK exit node + LAN subnet router<br/>Pi-hole v6 + Unbound - ad-block + private DNS<br/>Docker, monitoring, dashboard, hardening"]
  end
  subgraph MUM["Mumbai home"]
    SP["Spearhead India broadband (~50 Mbps)<br/>(residential IP)"]
    Lap["Old laptop, Windows<br/>Tailscale: INDIA exit node<br/>always-on, lid closed"]
  end
  TS(("Tailscale control plane<br/>+ DERP relays"))
  Dev["All 9 family devices<br/>phones, laptops, TV"]
  Pi --> CF
  Lap --> SP
  CF -.-> TS
  SP -.-> TS
  Dev -.-> TS
  Pi ---|WireGuard mesh| Lap
  Dev -->|use London exit| Pi
  Dev -->|use India exit| Lap
```

_Figure 1 - System topology (two-site Tailscale mesh). Each device picks an exit
node: India OTT / India sites -> Mumbai; UK IP / London LAN -> London (Pi);
ad-blocking only -> no exit node (Pi-hole DNS still applies). Detailed views in
section 9._

Cost: GBP 0/month. Tailscale free Personal = 6 users + unlimited devices (fits
4 users / 9 devices). All other software is FOSS.

------------------------------------------------------------------------------
## 3. Component stack (all free / open source)
------------------------------------------------------------------------------
London Pi 5 (Raspberry Pi OS Bookworm 64-bit - to confirm):
- Tailscale (mesh, exit node, subnet router, MagicDNS)
- Pi-hole v6 (ad/tracker blocking) + Unbound (recursive DNS)
- Docker + Compose (to run Pi-hole/Unbound/monitoring cleanly)
- Monitoring: Uptime Kuma + Netdata (optional, "maximum features")
- Homepage dashboard (optional)
- Hardening: SSH key-only, fail2ban, unattended-upgrades

Visual: see Figure 2 (section 9) for the full service-stack diagram.

Mumbai old laptop (Windows):
- Tailscale only, advertised as exit node. See docs/MUMBAI-SETUP.md.

Clients: Tailscale apps on iOS, Android, Windows, TV.

------------------------------------------------------------------------------
## 4. Decisions locked (from user)
------------------------------------------------------------------------------
- Mumbai node = the old laptop, kept always-on. GBP 0. (docs/MUMBAI-SETUP.md)
- Mumbai ISP = Spearhead India (~50 Mbps, upload upgradable on request).
  What to request: see docs/ISP-PARTNER-REQUEST.md (mainly upload speed +
  confirm residential/non-datacenter IP + unlimited data).
- Mesh = Tailscale free (chosen; sidesteps CGNAT on both ends, free, fits users).
- London deployment = I drive the Pi over SSH from the London PC (same flat).
- DNS = Pi-hole + Unbound (ad-blocking + privacy), pushed to all clients.

------------------------------------------------------------------------------
## 5. Cost summary
------------------------------------------------------------------------------
| Item                              | Cost            | Notes                              |
|-----------------------------------|-----------------|------------------------------------|
| Tailscale (mesh)                  | GBP 0/mo        | Free Personal: 6 users, unlimited  |
| WireGuard / Pi-hole / Unbound     | GBP 0           | FOSS                               |
| London Pi 5                       | GBP 0           | Already owned                      |
| Mumbai node (old laptop)          | GBP 0           | Repurposed                         |
| Community Fibre public IP         | NOT needed      | Skip the GBP 4/mo add-on           |
| True anonymity (no-log VPN)       | OPTIONAL        | Only if Goal 4 matters; ~GBP 3-5/m |

Running cost: GBP 0/month. Optional anonymity layer is the only possible spend.

------------------------------------------------------------------------------
## 6. Deployment approach
------------------------------------------------------------------------------
- London: I SSH into the Pi from the London PC (same flat), build the stack,
  switch to SSH-key auth, then also reach it via its Tailscale IP from anywhere.
- Mumbai: the helper follows docs/MUMBAI-SETUP.md; I verify the egress IP and
  enable the exit node from the Tailscale admin console.
- Clients: install Tailscale per docs/CLIENT-SETUP.md (to be written), pick exit.

------------------------------------------------------------------------------
## 7. Build phases (checklist)
------------------------------------------------------------------------------
Visual summary: see Figure 7 in section 9.

Phase 0 - Planning & decisions [DONE]
  [x] Research architecture
  [x] Identify CGNAT + OTT constraints
  [x] Confirm Mumbai node (old laptop), ISP (Spearhead), helper, PC location
  [x] Write Mumbai runbook + ISP request list
  [ ] Get SSH access to London Pi + confirm OS (awaiting user)

Phase 1 - London Pi foundation
  [ ] OS update / hardening / SSH key-only auth
  [ ] Install Docker + Compose
  [ ] Install Tailscale, join tailnet, advertise UK exit node + LAN subnet router
  [ ] Deploy Pi-hole v6 + Unbound (Compose)
  [ ] Point tailnet DNS at Pi-hole (ad-block for all clients)
  [ ] Monitoring (Uptime Kuma / Netdata) + Homepage dashboard (optional)

Phase 2 - Clients
  [ ] Install Tailscale on 4 phones, 3 laptops, TV
  [ ] Configure exit-node switching per use case
  [ ] Verify UK exit + LAN access + ad-blocking

Phase 3 - Mumbai India exit node
  [ ] Helper runs docs/MUMBAI-SETUP.md (laptop on, Tailscale, advertise exit)
  [ ] Verify egress IP is residential Indian (not datacenter)
  [ ] Test JioHotstar / Airtel Xstream / HP Gas / Adani Electricity from London
  [ ] (If IP flagged datacenter) request residential IP from Spearhead partner

Phase 4 - Hardening & polish
  [ ] fail2ban, unattended-upgrades, backups, UPS consideration
  [ ] Family runbook (how to switch exit nodes; what to do if it stops)

------------------------------------------------------------------------------
## 8. Caveats (expectations)
------------------------------------------------------------------------------
- Anonymity: routing through your own Mumbai node hides London traffic from
  Community Fibre but exposes it to the Mumbai ISP (and vice versa). Self-hosting
  is NOT the same as a no-log commercial VPN. Optional layer if needed (~GBP 3-5/m).
- OTT account: JioHotstar still needs an Indian mobile number for OTP. User has
  Mumbai subscriptions, so fine.
- OTT quality: capped by Mumbai UPLOAD speed (1080p ~8 Mbps up, 4K ~25 Mbps up).
- OTT routing: while watching, the device must use the Mumbai exit node as a FULL
  tunnel (not DNS-only), because Jio checks IP + region on the stream itself.
- VPN/OTT use may conflict with OTT terms of service; this is the user's own
  paid subscriptions on their own residential IP.

------------------------------------------------------------------------------
## 9. Architecture & workflow diagrams (Mermaid)
------------------------------------------------------------------------------
These diagrams render on GitHub. Figure 1 (system topology) is inline in
section 2; the views below expand it - component stack, DNS chain, mesh
formation, routing decision, OTT streaming path, and the build roadmap.

### Figure 2 - London Pi component & service stack

```mermaid
flowchart TB
  Internet(("Internet / DNS root"))
  subgraph Pi5["Raspberry Pi 5 - Raspberry Pi OS Bookworm 64-bit"]
    TS["Tailscale<br/>WireGuard mesh, UK exit node,<br/>LAN subnet router, MagicDNS"]
    subgraph DK["Docker + Compose"]
      PH["Pi-hole v6 - ad/tracker blocking"]
      UB["Unbound - recursive DNS resolver"]
      KU["Uptime Kuma - uptime monitoring"]
      ND["Netdata - system metrics"]
      HP["Homepage - dashboard"]
    end
    SSH["sshd - key-only auth"]
    F2B["fail2ban"]
    UP["unattended-upgrades"]
  end
  Clients(["Tailnet clients"]) --> TS
  TS -->|tailnet DNS| PH
  PH -->|forward| UB
  UB -->|recursive resolve| Internet
```

### Figure 3 - DNS resolution + ad-blocking chain

```mermaid
sequenceDiagram
  participant C as Client device
  participant TS as Tailscale MagicDNS
  participant PH as Pi-hole v6
  participant UB as Unbound
  participant R as DNS root servers
  C->>TS: query ads.tracker.com
  TS->>PH: forward - tailnet DNS is Pi-hole
  alt on blocklist
    PH-->>C: return 0.0.0.0 - ad dropped
  else allowed domain example.com
    PH->>UB: resolve example.com
    UB->>R: iterative lookup
    R-->>UB: IP answer
    UB-->>PH: IP address
    PH-->>C: IP address - no leak to ISP or Google
  end
```

### Figure 4 - CGNAT traversal / mesh formation

```mermaid
sequenceDiagram
  participant Pi as London Pi
  participant Lap as Mumbai laptop
  participant CP as Tailscale control plane
  participant DERP as DERP relay
  Pi->>CP: login - advertise UK exit + LAN subnet
  Lap->>CP: login - advertise India exit
  CP-->>Pi: peer list + discovered endpoints
  CP-->>Lap: peer list + discovered endpoints
  Pi->>Lap: NAT traversal - STUN / UDP hole-punch
  alt direct UDP path established
    Pi->>Lap: direct WireGuard tunnel - preferred
  else both behind CGNAT or blocked
    Pi->>DERP: relay connect
    Lap->>DERP: relay connect
    DERP-->>Pi: relayed WireGuard - still E2E encrypted
  end
```

### Figure 5 - Traffic routing decision (which exit node)

```mermaid
flowchart TD
  S(["Device wants to reach a site or stream"]) --> Q1{"India OTT or India-only site?<br/>JioHotstar, Airtel Xstream, HP Gas"}
  Q1 -->|yes| M["Use MUMBAI exit node - full tunnel"]
  Q1 -->|no| Q2{"UK IP or reach London home LAN?"}
  Q2 -->|yes| L["Use LONDON Pi exit node"]
  Q2 -->|no| Q3{"Just want ad-blocking, no location change?"}
  Q3 -->|yes| O["Exit node OFF - Pi-hole DNS still applies"]
```

### Figure 6 - OTT streaming path (London watches via Mumbai)

```mermaid
sequenceDiagram
  participant You
  participant D as London device
  participant Lap as Mumbai laptop
  participant Jio as JioHotstar
  You->>D: play title - exit node is Mumbai
  D->>Lap: full-tunnel via WireGuard
  Lap->>Jio: HTTPS from residential Indian IP
  Jio->>Jio: geo-check - residential India IP + region PASS
  Jio-->>Lap: video stream
  Lap-->>D: stream over WireGuard
  D-->>You: 1080p / 4K playback
  Note over Lap: quality capped by Mumbai UPLOAD speed - 1080p ~8 Mbps, 4K ~25 Mbps
```

### Figure 7 - Build phase roadmap

```mermaid
flowchart LR
  P0["Phase 0 - DONE<br/>research + architecture +<br/>Mumbai runbook + ISP list"]
  P1["Phase 1 - London Pi<br/>BLOCKER: SSH access to Pi"]
  P2["Phase 2 - clients"]
  P3["Phase 3 - Mumbai exit"]
  P4["Phase 4 - hardening + polish"]
  P0 --> P1 --> P2 --> P3 --> P4
  classDef done fill:#d4edda,stroke:#28a745,color:#000
  classDef now fill:#fff3cd,stroke:#ffc107,color:#000
  class P0 done
  class P1 now
```

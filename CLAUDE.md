# RNR-VPN - Project Notes for Claude Code

Self-hosted, near-zero-cost personal VPN mesh spanning a London flat and a Mumbai
home. Product / project name: **RNR-VPN** (hyphenated). Repo folder is also
`RNR-VPN` (hyphenated) - both match. History: on 2026-07-14 this repo was moved
from Windows (C:\Users\vince\Projects\RNR VPN, with a space) to WSL2/Ubuntu
(/home/vincentspereira/Projects/RNR-VPN, hyphenated), and the folder was renamed
to match the project name. The old "spaced folder on purpose" convention no longer
applies - do not reintroduce the space.

Owner: Vincent S. Pereira. Status: building (planning complete, deployment in
progress). Current blocker: SSH access to the London Pi to start Phase 1.

## Read these first
- `README.md` - user-facing overview: goals, cost, roadmap, caveats.
- `ARCHITECTURE.md` - full design, locked decisions, build-phase checklist, and
  Mermaid architecture / workflow diagrams (section 9).
- `docs/MUMBAI-SETUP.md` - plain-language runbook for the Mumbai helper.
- `docs/ISP-PARTNER-REQUEST.md` - what to ask the Spearhead (Mumbai) ISP partner.

These four docs are the source of truth. Keep them mutually consistent; when a
decision changes, update every reference.

## The two non-negotiable constraints (do NOT re-litigate)
1. **Community Fibre (London) is behind CGNAT.** No public IPv4, no port forwarding.
   -> Tailscale (WireGuard mesh) is mandatory. Do NOT propose classic WireGuard +
   port-forwarding, and do NOT suggest paying for the GBP 4/mo public-IP add-on.
   Resolved.
2. **Indian OTT (JioHotstar, Airtel Xstream) blocks all datacenter / VPS /
   commercial-VPN IPs.** -> The India exit node MUST egress from a residential Indian
   IP on the Mumbai home's own broadband. A cheap India VPS will NOT work. Resolved.

## Architecture (two-node Tailscale mesh)
- **London Pi 5** (Raspberry Pi OS Bookworm 64-bit) = hub: Tailscale UK exit node +
  LAN subnet router, Pi-hole v6 + Unbound (ad-blocking + private recursive DNS),
  Docker + Compose, monitoring (Uptime Kuma + Netdata), Homepage dashboard,
  hardening (SSH key-only, fail2ban, unattended-upgrades).
- **Mumbai old laptop** (Windows) = India exit node on a residential IP. Runs
  Tailscale only, advertised as an exit node.
- **Clients**: Tailscale apps on iOS, Android, Windows, and the TV. Each device picks
  an exit node on demand.
- All inter-node traffic is end-to-end encrypted (WireGuard), even via relays.

## Cost framing
Running cost is **GBP 0/month**: Tailscale free Personal (6 users / unlimited
devices), all other software FOSS, hardware already owned. The only optional spend is
a no-log commercial VPN (~GBP 3-5/m) IF true ISP-blind anonymity is wanted - which
self-hosting alone does NOT provide. Preserve this framing in docs and decisions.

## Repository layout
```
RNR-VPN/
  README.md                 User-facing overview (goals, cost, roadmap, caveats).
  ARCHITECTURE.md           Full design, locked decisions, build-phase checklist.
  CLAUDE.md                 This file - guidance for Claude Code.
  .gitignore                Excludes secrets, Tailscale/WG state, Pi-hole/Unbound data.
  docs/
    MUMBAI-SETUP.md         Plain-language runbook for the Mumbai helper.
    ISP-PARTNER-REQUEST.md  What to ask the Mumbai ISP partner for.
```
More files (compose files, install scripts, `docs/CLIENT-SETUP.md`) will be added as
the build progresses. When you add one, update this layout AND README.md section 7.

## Conventions for this repo
- **ASCII only.** No Unicode symbols (checkmarks, arrows, emojis). The existing docs
  are plain ASCII; match them. Windows console (cp1252) breaks on Unicode.
- **GBP costs**, not USD/INR. Hardware that is "already owned" counts as GBP 0.
- **No secrets in git.** `.gitignore` already excludes `.env`, `*.key`, `*.pem`,
  `wg*.conf`, `wg*.key`, `tailscale-state/`, `pihole/etc/`, `pihole/data/`,
  `unbound/data/`. The Mumbai runbook uses the placeholder `PASTE-CODE-HERE` for the
  one-time Tailscale auth key - never inline a real authkey.
- **Tone**: plain language, especially in `docs/MUMBAI-SETUP.md` (the Mumbai helper is
  non-technical). Numbered steps, no jargon.
- **Integrate, do not build crypto.** Use Tailscale, Pi-hole, Unbound, WireGuard
  as-is. Do NOT roll a custom VPN / crypto stack.

## Build roadmap (current state)
- [x] Phase 0 - Research, lock architecture, write Mumbai runbook + ISP request list.
- [ ] Phase 1 - London Pi: Docker, Tailscale (UK exit + LAN subnet router), Pi-hole v6
      + Unbound, mesh DNS, monitoring. **Blocker: SSH access to the London Pi.**
- [ ] Phase 2 - Clients: install Tailscale on phones / laptops / TV; verify UK exit,
      LAN access, ad-blocking.
- [ ] Phase 3 - Mumbai India exit node: laptop online, verify residential Indian
      egress IP, test OTT / India sites from London.
- [ ] Phase 4 - Hardening + polish: fail2ban, unattended-upgrades, backups, UPS,
      family runbook.

When a phase completes, tick it here AND in README.md section 9 AND ARCHITECTURE.md
section 7 - keep all three in sync.

## Key facts
- London ISP: Community Fibre (CGNAT; public IP is an unneeded paid add-on).
- Mumbai ISP: Spearhead India (~50 Mbps; upload upgradable on request; confirm the
  public IP is residential, not datacenter).
- Tailnet: Tailscale free Personal (6 users, unlimited devices).
- Mumbai node = the old always-on Windows laptop; lid closed, "do nothing" on close,
  never sleeps.

## References
- Tailscale: https://tailscale.com/
- WireGuard: https://www.wireguard.com/
- Pi-hole: https://pi-hole.net/
- Unbound: https://nlnetlabs.nl/projects/unbound/about/
- PiVPN: https://www.pivpn.io/

# What to ask your Spearhead (Mumbai ISP) partner for

Bottom line first: with Tailscale, the Mumbai connection needs NO special
network setup. CGNAT, no public IP, no port forwarding - all fine; Tailscale
works through all of it. So there is nothing the partner MUST change for this
to function.

But because you can ask for changes, here is what actually helps OTT streaming,
in priority order.

------------------------------------------------------------------------------
MUST HAVE (ask for these)
------------------------------------------------------------------------------
1. ENOUGH UPLOAD SPEED (most important).
   When Vincent watches JioHotstar in London, the video flows OUT of Mumbai,
   so Mumbai's UPLOAD speed is the limit.
     - 1080p video needs ~8 Mbps upload.
     - 4K video needs ~25 Mbps upload.
   Current line is ~50 Mbps (probably download). Ask the partner to confirm/
   raise the UPLOAD to at least 25-50 Mbps. A symmetric 50/50 (or higher) is
   ideal.

2. UNLIMITED / high monthly data.
   Streaming is data-heavy (1 hour of HD ~ 1-3 GB). Confirm there is no low
   monthly cap. Business lines are usually fine - just confirm.

3. CONFIRM THE PUBLIC IP IS A NORMAL INDIAN-ISP ADDRESS (not a "data center").
   JioHotstar blocks data-center / hosting-company IPs. Spearhead is a real
   ISP so the IP is probably already fine, but ask the partner to confirm the
   public IP shows as "Spearhead / residential India" and NOT as a hosting/
   cloud company. Vincent will verify this with a quick online check once the
   laptop is online. If it IS flagged as data-center, the partner can likely
   move you to a residential-style IP range - that is the one thing that would
   genuinely matter for OTT.

------------------------------------------------------------------------------
NICE TO HAVE (optional, not required)
------------------------------------------------------------------------------
4. A STATIC public IP - useful only if we later want a direct connection as a
   backup. Not needed for the main plan.
5. Removing CGNAT / giving a real public IP - a bonus, not required.

------------------------------------------------------------------------------
NOT NEEDED (you can ignore)
------------------------------------------------------------------------------
- Port forwarding. Not required (Tailscale handles reachability).
- Any special router config.

------------------------------------------------------------------------------
ONE QUICK THING TO ASK THE PARTNER TO SEND YOU
------------------------------------------------------------------------------
Ask them to run a speed test from the Mumbai connection
(https://fast.com or https://speedtest.net) and send you the result - mainly
the UPLOAD number. Forward that to Vincent so we know what streaming quality
to expect (1080p vs 4K).

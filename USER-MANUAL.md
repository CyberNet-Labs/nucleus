# Nucleus User Manual

Nucleus is CyberNet Labs' client app for CyberGuard, our network and web security toolkit — plus
Lumen and Flux, which aren't built yet. This manual covers what's here today: how to install it,
how each CyberGuard tool works, and how it differs slightly across macOS, Windows and Linux.

This is a **beta**. Some things are rough around the edges, and a few features work slightly
differently between platforms — noted throughout.

---

## 1. Installing and opening Nucleus

Download the build for your platform from this release. All builds are currently unsigned, so
your operating system will warn you the first time you open one — that's expected for a beta.

### macOS
1. Unzip `Nucleus-macOS-universal.zip`. It works on both Apple Silicon and Intel Macs — one file,
   no need to pick the right version.
2. **Right-click the app → Open**, then confirm. (Double-clicking will just show a warning and
   refuse to open — right-click → Open is what tells macOS you trust it.)

### Windows
1. Download `Nucleus-windows-x64.exe`. It's self-contained — no separate .NET install needed.
2. Windows SmartScreen will likely show a warning ("Windows protected your PC"). Click
   **More info**, then **Run anyway**.

### Linux
1. Download `Nucleus-linux-x64`.
2. Make it executable and run it:
   ```
   chmod +x Nucleus-linux-x64
   ./Nucleus-linux-x64
   ```
3. You need a desktop session (X11 or Wayland) — this is a graphical app, not a command-line tool.

---

## 2. Finding your way around

Every screen has the same shape: a dropdown in the title bar lets you switch between **Overview**
and each service (currently just **CyberGuard** — Lumen and Flux are placeholder screens for now).

Inside CyberGuard, a second dropdown labeled **Select Scan Type** switches between the five tools
described below. Whichever one you pick, the same three-part layout applies:

1. **Controls** — enter a target, choose options, and an acknowledgment checkbox.
2. **The acknowledgment checkbox** — every active scan requires you to confirm *"I own this host
   (or network) or have written permission to scan it."* This isn't optional, and it's there for a
   real reason: port scanning and network sweeps can look like an attack to the system you're
   scanning, and running them against something you don't own or have permission to test can be
   illegal depending on where you are. Only scan things you're actually allowed to.
3. **Findings** — results appear here once a scan starts, updating live as it runs.

On the Mac, host discovery and MAC-address lookup work on macOS itself; Windows and Linux have
their own equivalents (details in each tool's section below). None of these tools work on iOS or
iPadOS, since Apple doesn't let apps read other devices' network details there.

**The profile icon**, next to the copyright label in the title bar, opens a small panel for a name
and email. There's no beta-tester account system yet, so this is stored only on your own device —
it isn't synced anywhere. It's there so the panel's design can be tried out now; a real signed-in
account will use the same spot later.

---

## 3. The network status card

At the top of the CyberGuard screen you'll see your device's **IP address**, its **subnet**, and
whether a **VPN** is active. If your subnet is larger than a `/24` (256 addresses) — common on
office or ISP-shared networks — the card also suggests the specific `/24` slice to actually scan,
since Host Discovery can't sweep a whole huge network at once. Host Discovery's target field
pre-fills with that suggestion automatically.

**If you have more than one** of anything here — two Wi-Fi adapters, a Wi-Fi and an Ethernet
connection both up, or two VPNs connected at once — a small row of toggle chips appears under that
item, letting you switch which one is shown (and, for your IP/subnet, which one Host Discovery's
suggestion is based on).

**Wi-Fi Pentest Adapter row:** shown above the IP/Subnet/VPN row, on Port Scan, Host Discovery,
Firewall Scan and Advanced Scan (not on Web Security, since that doesn't touch your local network
hardware). It checks connected USB devices against a curated list of Wi-Fi chipsets known to
support monitor mode/packet injection — the kind used with tools like aircrack-ng — and reports:

- **Pentest-capable** — a known chipset is connected, with its name.
- **Adapter connected** — something from a known Wi-Fi chipset vendor is connected, but the exact
  model isn't in the list.
- **None detected** — no such adapter found.

This is detection only: it identifies hardware that's present, it doesn't enable monitor mode or
do any capture/injection itself.

---

## 4. Port Scan

**What it does:** checks a list of ports on one host to see which are open, which service is
likely running on each, and — for a handful of common ports — reads the greeting the service sends
back (its "banner"). It also flags specific ports known to be risky to leave exposed (Telnet, RDP,
unauthenticated databases like Redis, and so on).

**Target field pre-fills automatically** with your network's router/gateway address (e.g.
`192.168.1.1`) when one can be detected — a convenient starting point since it's always reachable.
This doesn't mean it's automatically fine to scan: on a network you don't administer (office
Wi-Fi, a café), the router isn't necessarily yours to test just because it's convenient. Type over
it with your actual target, and only proceed once the ownership checkbox is genuinely true.

**How to use it:**
1. Type a hostname or IP address, e.g. `scanme.nmap.org` — a host the nmap project runs
   specifically so people can test scanners like this one safely.
2. Pick a profile: **Quick** (the ~30 most common ports), **Standard** (1–1024), **Extended**
   (1–5000), or **Custom** — enter your own list, e.g. `22,80,443,8000-8100` (comma-separated ports
   and/or ranges, capped at 10,000 ports). Quick is almost always the right first choice.
3. Each profile shows a time estimate, e.g. "30 ports · up to ~2s worst case." This is a worst-case
   upper bound, not a prediction — it's how long the scan takes if the host answers nothing at all.
   A responsive host is usually much faster.
4. Tick the ownership/permission checkbox, then **Start scan**.

You'll see a live count of ports scanned and ports found open, then a table of just the open ports
— port number, state, service name, and response time. If a risky service turns up (like an open
database port), a warning appears underneath explaining why it matters.

Once a scan finishes, a **Check firewall behavior on this host** link hands the same host straight
to Firewall Scan — see §7 for why you'd want both.

**What it can't do:** this is a plain TCP connect scan — the same technique nmap calls `-sT`. It
can't do stealth SYN scans or guess the target's operating system, both of which need direct
access to raw network packets that a regular desktop app isn't allowed.

---

## 5. Host Discovery

**What it does:** sweeps every address in a network range and reports which ones respond, along
with each device's MAC address, manufacturer (Apple, Samsung, TP-Link, etc. — looked up from the
official IEEE hardware registry), and a name if the device offers one. It also remembers what it's
found: on a later sweep of the same range, devices that used to respond but have gone quiet show
up as **Offline**, with how long ago they were last seen.

**How to use it:**
1. Enter a network in CIDR form, e.g. `192.168.1.0/24` — the field pre-fills with a sensible guess
   based on your own network (see §3). The prefix must be `/24` through `/32` — at most 256
   addresses per sweep, a deliberate limit.
2. Optionally tick **Also scan for open ports on each device found** — Host Discovery and Port Scan
   then work together in one run: every device that answers also gets a Quick-profile port scan
   (one device at a time, after the sweep finishes), so you get devices *and* their open ports
   without switching tools.
3. Tick the ownership checkbox, then **Start sweep**.
4. Watch devices appear as they respond. When the sweep finishes, any previously-seen devices in
   that range that didn't answer this time show up too, marked Offline. If you turned on the port
   toggle, each device's open ports (or "No open ports found") appear underneath it once its scan
   completes — watch the status chip switch to "Scanning ports" during that phase.

A **Clear device history** button resets what's remembered, if you want a fresh start.

**Why some devices show "Unknown device" or "MAC unavailable":**
- A name only appears if the device actually announces one (over Bonjour/mDNS, NetBIOS, or a
  router-provided reverse-DNS entry) — many phones and IoT devices don't.
- A MAC address showing as **"private Wi-Fi address"** instead of a manufacturer name means that
  device is using a random per-network address (a privacy feature most phones and laptops use by
  default) — the manufacturer genuinely can't be determined from it.
- If you're on a VPN, it can occasionally intercept local-network traffic and interfere with this
  scan. macOS actively excludes VPN interfaces from local scans; Windows and Linux do a best-effort
  version of the same fix (details in each platform's own documentation in the private repo) — it
  helps in the common case but isn't a hard guarantee on those two.

---

## 6. Web Security

This tool has two parts, both working against a website you name.

### Web Security Headers

**What it does:** loads a website once and checks for six HTTP response headers that protect
visitors — things like forcing HTTPS on return visits, restricting where scripts can load from,
and preventing the page from being embedded in a hidden frame on another site (clickjacking).

**How to use it:**
1. Enter a website, e.g. `cybernetlabs.cc` (no need to type `https://` — it's added automatically).
2. **Check headers.**

You'll get a score (e.g. "4 of 6 present") and a line-by-line breakdown — a checkmark and the
header's actual value if it's present, or an explanation of what that header is for if it's
missing.

### Web App Security

**What it does:** checks a page you name for authentication bypass and session cookie hygiene,
using plain unauthenticated HTTP requests. There's no browser engine in the app, so it can't drive
an actual login form — this is a set of heuristic checks, not a full penetration test of your login
flow.

**How to use it:**
1. Enter a page you believe requires login, e.g. `example.com/dashboard`.
2. Tick **I own this web app or have written permission to test it.**
3. **Check web app security.**

You'll get three things:
- **Access check** — whether the page returned content directly (a possible auth bypass, flagged
  for you to verify manually), was blocked (401/403), or redirected somewhere that looks like a
  login page.
- **Cookie flags** — every cookie the response set, with its Secure/HttpOnly/SameSite flags. Cookies
  that look session-related (by name) are marked "Session-like," and missing flags on those are
  highlighted.
- **Session rotation note** — the app visits the page twice, in two completely separate
  unauthenticated sessions, and checks whether session-looking cookies get a fresh value each time.
  An identical value both times can indicate a weak/predictable session ID. This is a precondition
  check, not a full session fixation test — a real one needs to trace the ID through an actual
  login, which this tool doesn't automate.

---

## 7. Firewall Scan

**How this differs from Port Scan:** Port Scan (§4) tells you *which ports are open and what's
running on them*. Firewall Scan asks a different question — *how is this host's firewall behaving*
— by looking at *how* each port answered (open, refused, rejected, or dropped) to work out what
kind of filtering is in place. Same underlying technique, different question. Target field
pre-fills with your router's address, same as Port Scan (see §4's note on that) — and once a scan
finishes, a **Run a full port scan on this host** link hands the host over to Port Scan, so you can
move between "what's open" and "how's it filtered" on the same target without retyping anything.

**What it does:** gives you a plain-language verdict:

- **"No firewall filtering seen"** — every port answered, one way or another. Nothing is quietly
  blocking traffic to this host.
- **"Firewall rejecting traffic"** — blocked ports send back an explicit "unreachable" error
  instead of staying silent (a reject policy).
- **"Firewall detected"** — a mix of silence and answers, typical of a default-deny firewall.
- **"No ports answered"** — either the host is offline, or it's in full stealth mode.

**How to use it:**
1. Enter a host, pick a profile, tick the checkbox.
2. Optionally switch on **Also test outbound rules** — this checks which ports *your own* network
   lets you connect *out* on, by testing against `portquiz.net` (a public service built exactly
   for this — it accepts connections on every port, so anything that fails is being blocked by
   your network, not the far end).
3. **Start firewall scan.**

You'll see counts of Open / Refused / Rejected / Dropped ports, the verdict, and — if you turned
on the outbound test — a second list showing which outbound ports your network allows.

---

## 8. Advanced Scan

**What it does:** combines the features above into one run, with each piece switched on or off
individually:

- **Find devices on a network** — the Host Discovery sweep
- **Name and MAC address** — device info for whatever's found
- **Scan ports** — with sub-options for banners and risk checks, same as Port Scan
- **Web security headers** — checked against any web server found on a discovered device

**How to use it:** switch on whichever combination you want, fill in the target (a network range
if "Find devices" is on, otherwise a single host), tick the checkbox, and **Start**. Results build
up per-device as the run progresses.

One limit worth knowing: if you combine "Find devices" with "Scan ports," the port list is capped
to the Quick profile — scanning every port on every device on a whole network would otherwise take
a very long time.

---

## 9. Scan Activity

Every scan you run — of any type — gets logged here: what kind, the target, a one-line summary,
and when it ran. It's local to your device and only lasts for the current session (it clears when
you quit the app). An **Export** button saves the whole log as a JSON file, and **Clear** empties
it.

---

## 10. Feature availability by platform

All three platforms share the same five CyberGuard tools and the same overall design. A few
specifics differ:

| | macOS | Windows | Linux |
|---|---|---|---|
| All 5 scan types | ✅ | ✅ | ✅ |
| Scan log + device history | ✅ | ✅ | ✅ |
| Device name lookup (Bonjour/NetBIOS/reverse DNS) | ✅ | ✅ | ✅ |
| VPN-safe local scanning | Guaranteed | Best-effort | Best-effort |
| Network status card, with multi-adapter/interface/VPN toggles | ✅ | ✅ | ✅ |
| Wi-Fi pentest-adapter detection | ✅ | ✅ | ✅ |
| Web App Security | ✅ | ✅ | ✅ |

---

## 11. Getting help

This is an early beta — expect rough edges. If something doesn't work as described here, that's
useful to know. CyberNet Labs owns and maintains Nucleus; this manual will be updated as the app
changes.

© 2026 CyberNet Labs. All rights reserved.

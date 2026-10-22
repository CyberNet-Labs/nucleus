# Nucleus User Manual

*For version 0.2.0.0 (public beta).*

Nucleus is CyberNet Labs' client app for CyberGuard, our network and web security toolkit — plus
Lumen and Flux, which aren't built yet. This manual covers what's here today: how to install it,
how each CyberGuard tool works, and how it differs slightly across macOS, Windows and Linux.

This is a **beta**. Some things are rough around the edges, and a few features work slightly
differently between platforms — noted throughout.

**Only scan networks, devices and websites you own or have written permission to test.** Scanning
something you're not authorized to test can be illegal, and it can look like an attack to the
system you're scanning.

---

## 1. Installing and opening Nucleus

Download the build for your platform from the latest release. The builds aren't signed with a
commercial certificate yet, so your operating system will warn you the first time you open one —
that's expected for a beta.

### macOS
1. Unzip `Nucleus-macOS-universal.zip`. It works on both Apple Silicon and Intel Macs.
2. **Right-click the app → Open**, then confirm. (Double-clicking just shows a warning and refuses
   to open — right-click → Open is what tells macOS you trust it.)

### Windows
1. Download `Nucleus-windows-x64.exe`. It's self-contained — no separate .NET install needed.
2. Windows SmartScreen will likely say "Windows protected your PC". Click **More info**, then
   **Run anyway**. The file is signed by CyberNet Labs (see Properties → Digital Signatures).

### Linux
1. Download `Nucleus-linux-x64`.
2. Make it executable and run it:
   ```
   chmod +x Nucleus-linux-x64
   ./Nucleus-linux-x64
   ```
3. You need a desktop session (X11 or Wayland) — this is a graphical app, not a command-line tool.

### Updates
From this version on, Nucleus checks for new versions when it starts. If one is available, a banner
offers to update: it downloads the new version, replaces the app and relaunches it. You can also
check by hand in **Settings → Software Update**.

---

## 2. Finding your way around

**Picking a tool.** CyberGuard's tools are chosen from one dropdown — **Select Scan Type** on
macOS, and the dropdown in the middle of the title bar on Windows and Linux. The tools are:
Overview, Port Scan, Host Discovery, Web Security, Firewall Scan, Advanced Scan and Packet Capture.
(On macOS, the title bar also switches between CyberGuard and the Lumen/Flux placeholder screens.)

**Easy and Advanced mode.** In **Settings** you can choose:
- **Easy** — shows only Overview, Port Scan, Host Discovery and Web Security. A good place to start.
- **Advanced** — shows every tool.

**Session authorization.** Near the top of CyberGuard there's one switch: *"I own these targets, or
have written permission to test them."* No scan can start until it's on. You confirm it once per
session and it covers every tool — each tool shows a short note saying whether you're covered.
You can add an optional **scope note** (for example, the client or engagement name). The time you
confirmed and your scope note are recorded with every scan in the Scan Activity log (§12). The
switch turns itself off every time the app restarts, so you confirm again each session.

**Every tool has the same layout:** controls at the top (target, options, Start), then
**Findings** below, updating live as the scan runs.

**The profile icon**, next to the copyright label in the title bar, opens a small panel for a name
and email. There's no account system yet, so this is stored only on your own device.

None of these tools work on iOS or iPadOS, since Apple doesn't let apps read other devices'
network details there.

---

## 3. The network status card

At the top of CyberGuard you'll see your device's **IP address**, its **subnet** and **gateway**
(your router), and whether a **VPN** is active.

**More than one network?** If you're connected to several at once — Wi-Fi plus Ethernet, or a
tagged VLAN — a **Connected networks** list shows each one with its own subnet and gateway. Toggle
chips let you choose which one is shown — and choosing one also makes every scan go out through
that network, so you can deliberately test through a specific VLAN or VPN.

**Wi-Fi Pentest Adapter row** (shown for Port Scan, Host Discovery, Firewall Scan and Advanced
Scan): checks connected USB devices against a list of Wi-Fi chipsets known to support monitor
mode/packet injection, and reports **Pentest-capable**, **Adapter connected** (known vendor, model
not in the list) or **None detected**. Detection only — it doesn't enable monitor mode or capture
anything.

---

## 4. Overview

**What it does:** one card per tool showing its latest result this session — for example "3 open
ports", "12 hosts found", "Firewall detected" or "Not run yet" — coloured green, amber or red by
how concerning it is. A summary line shows how many tools you've run. Click any card to jump to
that tool with its results still there.

**Export session report:** the Overview tab has a button that saves everything scanned this
session as a single HTML file — port scans (including full-subnet scans, host by host), discovered
devices, the firewall verdict and open ports, web header results and the web-app access check.
Open it in any browser, print it, or send it on.

---

## 5. Port Scan

**What it does:** checks a list of ports on a host to see which are open, which service is likely
running on each, and — for common ports — reads the greeting the service sends back (its
"banner"). It flags ports that are risky to leave exposed (Telnet, RDP, unauthenticated databases
like Redis, and so on).

**How to use it:**
1. Type a hostname or IP address, e.g. `scanme.nmap.org` — a host the Nmap project runs so people
   can test scanners safely. The field pre-fills with your router's address as a convenient
   starting point; that doesn't mean you're authorized to scan it.
2. Pick a profile: **Quick** (~30 common ports), **Standard** (1–1024), **Extended** (1–5000), or
   **Custom** — your own list, e.g. `22,80,443,8000-8100` (up to 10,000 ports). Each shows a
   worst-case time estimate.
3. Optionally switch on **Scan a full subnet instead of one host** and enter a network like
   `192.168.1.0/24`: every device that answers gets port-scanned, and results appear in a table you
   can expand host by host.
4. Make sure session authorization is on (§2), then **Start scan**.

You'll see live progress, then a table of the open ports with service, state and response time,
and a warning under any risky service. A **Check firewall behavior on this host** link hands the
same host to Firewall Scan (§8).

**What it can't do:** this is a plain TCP connect scan (what Nmap calls `-sT`). It can't do stealth
SYN scans or guess the operating system, which need raw packet access a regular app doesn't have.

---

## 6. Host Discovery

**What it does:** sweeps a network range and lists every device that responds, with its MAC
address, manufacturer (from the official IEEE registry) and name if it offers one. It remembers
what it found: on later sweeps, devices seen for the first time get a **New** badge, and devices
that have gone quiet show as **Offline** with when they were last seen.

**How to use it:**
1. Enter a network in CIDR form, e.g. `192.168.1.0/24` (pre-filled from your own network). Ranges
   up to a `/16` (65,536 addresses) are allowed, with a time estimate shown — bigger ranges take
   longer.
2. Optionally switch on **Also scan for open ports on each device found** to get each device's
   open ports in the same run.
3. **Start sweep.**

**Clear device history** resets what's remembered.

**Why some devices show "Unknown device" or "MAC unavailable":**
- A name only appears if the device announces one (Bonjour/mDNS, NetBIOS, or reverse DNS).
- **"Private Wi-Fi address"** means the device uses a random per-network address (a privacy
  feature on most phones and laptops), so the manufacturer can't be known.
- A VPN can interfere with local scans. macOS excludes VPN interfaces automatically; Windows and
  Linux do a best-effort version of the same.

---

## 7. Web Security

Two checks against a website you name.

### Web Security Headers
Loads the site once and checks six HTTP response headers that protect visitors — forcing HTTPS,
limiting where scripts load from, blocking clickjacking, and so on. Enter a site such as
`cybernetlabs.cc` (https is added automatically) and click **Check headers**. You get a score (e.g.
"4 of 6 present") and a line per header: its value if present, or what it's for if missing.

### Web App Security
Checks a page you name for authentication bypass and session-cookie hygiene, using plain
unauthenticated requests (it can't log in through a form). Enter a page you believe requires login,
e.g. `example.com/dashboard`, and click **Check web app security**. You get:
- **Access check** — whether the page returned content without logging in (a possible bypass to
  verify), was blocked (401/403), or redirected to a login page.
- **Cookie flags** — each cookie's Secure/HttpOnly/SameSite flags, with missing flags on
  session-like cookies highlighted.
- **Session rotation** — whether session-like cookies change between two separate visits. An
  identical value both times can mean a weak session ID.

---

## 8. Firewall Scan

**How it differs from Port Scan:** Port Scan asks *which ports are open*. Firewall Scan asks *how
is this host's firewall behaving*, by looking at how each port answers — **open**, **refused** (the
host said no), **rejected** (an "unreachable" error came back) or **dropped** (no reply at all).

**The verdict:**
- **"No firewall filtering seen"** — every port answered one way or another.
- **"Firewall rejecting traffic"** — blocked ports send back an explicit error (a reject policy).
- **"Firewall detected"** — a mix of silence and answers, typical of a default-deny firewall.
- **"No ports answered"** — the host is offline or fully stealthed.

**How to use it:**
1. Enter a host and pick a profile.
2. Optionally switch on **Also test outbound rules** — checks which ports *your* network lets you
   connect *out* on, by testing against `portquiz.net` (a public service that accepts connections
   on every port, so any failure is your network blocking it).
3. Optionally switch on **Scan a full subnet** to check every device on a network.
4. **Start firewall scan.**

**Changes since last scan:** if you've scanned the same host before, a box shows what changed —
ports newly blocked or newly opened since then. Firewall rules are policy, so a change is often the
most important thing to see.

A **Run a full port scan on this host** link hands the host to Port Scan.

---

## 9. Advanced Scan

Combines the tools into one run, each part switched on or off:
- **Find devices on a network** (Host Discovery)
- **Name and MAC address**
- **Scan ports**, with **Service banners** and **Risk checks**
- **Web security headers** on any web server found

Fill in the target (a network if "Find devices" is on, otherwise one host) and **Start**. Results
build up device by device. If you combine "Find devices" with "Scan ports", the port list is limited
to Quick, since scanning every port on every device would take very long.

---

## 10. Packet Capture

**What it does:** captures live network traffic on the interface you choose and saves it as a
standard `.pcap` file, which tools like Wireshark can open. While capturing, it shows each packet's
time, protocol, source, destination and size, colour-coded by protocol.

**Before your first capture each session**, Nucleus shows a warning: capturing traffic can record
other people's communications, and capturing traffic you don't own or aren't authorized to capture
can be a criminal offence (for example under South Africa's Cybercrimes Act, section 3, or the UK's
Computer Misuse Act). Choose **I understand, start capture** to continue. Your acknowledgement is
recorded in the Scan Activity log.

**How to use it:**
1. Pick an interface from the list.
2. **Start capture**; **Stop capture** when done. The file is saved to your chosen folder
   (Settings → Packet capture save location); a button opens that folder.
3. Use the **filter** box to narrow the list — a protocol (`tcp`, `udp`, `arp`), a port (`443`) or
   an address or text.
4. Select a packet and choose **Follow Stream** to see just that conversation.
5. **Statistics** show packet counts per protocol and the busiest conversations.

**Permissions, per platform:**
- **macOS** — the first time, macOS blocks access to capture devices. Click **Grant Access & Retry**
  and enter your password once.
- **Windows** — needs the free Npcap driver. If no interfaces are listed, click **Install Npcap**
  and Nucleus downloads and runs the official installer. You may also need to run Nucleus as
  Administrator.
- **Linux** — if capture isn't permitted, use the one-click permission fix the app offers (it asks
  for your password).

This shows packet headers, not decoded payloads — for deep protocol analysis, open the saved
`.pcap` in Wireshark.

---

## 11. Settings

Open with the gear button. You can set:
- **Mode** — Easy or Advanced (§2).
- **Default scan type** — which tool opens first.
- **Default port scan profile** — Quick, Standard or Extended.
- **Packet capture save location**.
- **Software Update** — current version, and **Check for Updates**.

---

## 12. Scan Activity

Every scan of any type is logged: what kind, the target, a one-line summary and when it ran — plus
when you confirmed session authorization and your scope note, if you gave one. The log stays on your
device and clears when you quit the app. **Export** saves it as a JSON file; **Clear** empties it.

---

## 13. Feature availability by platform

| | macOS | Windows | Linux |
|---|---|---|---|
| All seven CyberGuard tools (incl. Overview and Packet Capture) | ✅ | ✅ | ✅ |
| Full-subnet Port Scan and Firewall Scan | ✅ | ✅ | ✅ |
| Session authorization and capture warning | ✅ | ✅ | ✅ |
| Session report export (HTML) | ✅ | ✅ | ✅ |
| Automatic updates | ✅ | ✅ | ✅ |
| Scan log and device history | ✅ | ✅ | ✅ |
| Network status card with multiple networks/VPNs | ✅ | ✅ | ✅ |
| Wi-Fi pentest-adapter detection | ✅ | ✅ | ✅ |
| Pin scans to a chosen network | ✅ | ✅ | ✅ |
| VPN-safe local scanning | Guaranteed | Best-effort | Best-effort |
| Lumen/Flux placeholder screens | ✅ | — | — |

---

## 14. Getting help

This is a beta — expect rough edges. If something doesn't work as described here, that's useful to
know. CyberNet Labs owns and maintains Nucleus; this manual is updated as the app changes.

© 2026 CyberNet Labs. All rights reserved.

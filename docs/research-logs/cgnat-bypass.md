# 🕵️ Research Log: The Evolution of Connectivity

**Objective:** Host a private cloud and media server behind ISP-grade NAT (CGNAT).
**Tools Used:** Claude 3.5 Sonnet, Ubuntu Terminal, Cron, WireGuard.

---

## 1. Attempt #1: The "Manual" Way (DuckDNS + WireGuard)
**Status:** 🔴 Abandoned (Too Fragile)

Before discovering Zero Trust tunnels, I attempted to build a traditional VPN access point using Dynamic DNS.

### The Scripting
I wrote a custom Bash script hooked into `cron` to update my dynamic IP every 5 minutes.

![Cron job for DuckDNS updates](./images/duckDnsApproach.webp)
*(Fig 1: The manual `cron` job used to sync dynamic IPs with DuckDNS - **Terminal Output**)*

### The Failure Point
I tried to pair this with a raw WireGuard installation. However, because my ISP puts me behind a double NAT (CGNAT), the incoming UDP packets for the WireGuard handshake were constantly dropped.

<img src="./images/claudeChat_0.png" width="450" alt="Claude guiding WireGuard setup">
<br>
*(Fig 2: Attempting to configure raw WireGuard keys)*

---

## 2. Attempt #2: The "Tunnel" Breakthrough (Cloudflare)
**Status:** 🟢 Success

Realizing that *incoming* connections were blocked, I pivoted to "Reverse Tunneling" technologies that rely on *outbound* connections.

### Exploring Options
I consulted AI to find alternatives to port forwarding. We evaluated Ngrok, ZeroTier, and Cloudflare.

<img src="./images/claudeChat_1.png" width="450" alt="Asking Claude about CGNAT bypass options">
<br>
*(Fig 3: Exploring the concept of "Reverse Tunneling")*

### The ZeroTier Pivot
I initially set up a mesh network using ZeroTier to bypass the router restrictions entirely.

<img src="./images/claudeChat_2.png" width="450" alt="Claude guiding the ZeroTier setup">
<br>
*(Fig 4: Configuring the initial mesh network)*

### The Final Cloudflared Tunnel Implementation  
The final fix came in the form of the implementation of cloudflare tunnel using my custom domain name *patyux.me* using subdomains for different services mapped to their specific ports.
<img src="./images/tunnels.png" width="450" alt="Claude guiding the ZeroTier setup">

This lead to *REMOTE ACCESS* of important services shifting it from just a NAS to a full fledged Homelab. 
<br>
---

## 3. Debugging: The "Phantom File" Incident
**Status:** 🟢 Resolved

During the installation of the Nextcloud stack, I encountered a critical failure in the extraction process.

### The Error
The `tar` command failed with "not a bzip2 file" despite a successful download message.

![Terminal screenshot showing tar command failure](../Screenshots/debugging/failureInNextcloudInstallation.webp)
*(Fig 5: The breakdown of the installation script - **Terminal Output**)*

> **Note:** The `../` in the path above tells Markdown to go **up one folder** (out of `research-logs`) and then down into `Screenshots/debugging`.

### The Fix
I analyzed the file header in the terminal output. It was `[text/html]` with a size of only `5.62K`. The download link was actually a **webpage redirect**, not the file itself. I corrected the URL to point to the raw binary, solving the issue.

---

## 4. Product Polish: The "Ugly Error" Fix
**Status:** 🟡 Workaround Implemented

I wasn't satisfied with just "it works"; I wanted it to look professional. When the server was down, Cloudflare served a generic error page.

<img src="./images/claudeChat_3.png" width="450" alt="Discussing custom error pages with Claude">
<br>
*(Fig 6: Trying to engineer a custom "Maintenance Mode" page on the Free Tier)*

---

## 🧠 The Volume of Research

This infrastructure was built on the back of **46+ recorded debugging sessions** and hundreds of hours of trial and error.

<div align="center">
  <img src="./images/claudeConsole.png" width="400" alt="Search History showing 46 chats">
  <br>
  *(Fig 7: Knowledge Base)*
  <br><br>
  <img src="./images/claudeSidebar.png" width="250" alt="Sidebar showing diverse topics">
  <br>
  *(Fig 8: Scope of Work)*
</div>

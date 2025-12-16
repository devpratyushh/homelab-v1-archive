# 🏠 Ghar Labs v1: The "Genesis" Server (Legacy Archive)

![Status: Retired](https://img.shields.io/badge/Status-Retired-red?style=flat-square)
![Migrated To: TrueNAS](https://img.shields.io/badge/Migrated_To-TrueNAS_Scale-blue?style=flat-square)
![Host: Ubuntu](https://img.shields.io/badge/Host-Ubuntu_24.04_LTS-orange?style=flat-square)
![Uptime: 9 Days](https://img.shields.io/badge/Final_Uptime-9_Days-success?style=flat-square)

> **⚠️ ARCHIVE NOTICE:** This project is no longer active. It represents the "Proof of Concept" state of my homelab infrastructure from **July 2024 to July 2025**. All services and data have been migrated to a dedicated **TrueNAS Scale** machine (Ghar Labs v2) for ZFS data integrity and enterprise-grade redundancy.

---

## 📖 The Origin Story: Escaping the "Ecosystem Trap"

In **July 2024**, my digital life was fragmented across incompatible ecosystems. I was juggling an **iPad**, a **Windows PC**, and an **Android phone**. Moving a simple PDF or 4K video file between them was a friction-filled nightmare of cables and cloud upload limits.

I had a spare **500GB hard drive**, a virtualization host, and a simple goal: **Centralize the data.**

The project began as a basic NAS (Network Attached Storage). However, "scope creep" set in immediately. I realized that if the server was always on for file storage, it could also:
1.  **Automate Media:** Replace paid streaming services with a self-hosted "Netflix" (Jellyfin).
2.  **Sovereign Cloud:** Replace Google Drive with a private cloud (Nextcloud).
3.  **Secure Vault:** Host my own password manager (Vaultwarden).

Thus, **Ghar Labs** was born.

---

## 🏗️ Technical Architecture

This system ran as a monolithic **Ubuntu Server 24.04 LTS** virtual machine, orchestrating 10+ microservices via **Docker Compose**.

### Infrastructure Specs
* **Hypervisor:** Oracle VirtualBox (Windows Host).
* **Guest OS:** Ubuntu Server 24.04 LTS.
* **Resources:** 4 vCPUs, 4GB RAM.
* **Network:** Bridged Adapter Mode (Static IP: `192.168.29.5`).
* **Storage:** 500GB Virtual Disk (`.vDI`) mounted at `/mnt/data`.

### The "Access Strategy" (Bypassing CGNAT)
One of the biggest engineering challenges was my ISP's use of **CGNAT (Carrier-Grade NAT)**, which made the server invisible to the public internet (no public IP). Standard port forwarding was impossible.

We engineered a **Hybrid Access Model** to solve this:

| Access Type | Tool Used | Technology | Use Case |
| :--- | :--- | :--- | :--- |
| **Public Web** | **Cloudflare Tunnel** | `cloudflared` (Zero Trust) | Exposing HTTP services (Jellyfin, Nextcloud) to friends/family via `https://movies.gharlab.com`. |
| **Admin/File** | **Tailscale** | Mesh VPN (WireGuard) | Direct, high-speed SMB file transfers and SSH access from iPad/Laptop without exposing ports. |

---

## ⚙️ The "Unified Storage" Pattern

We avoided duplicate files and permission errors by pointing both the Downloader (qBittorrent) and the Player (Jellyfin) to the **exact same physical directory**.

**The Directory Structure:**
```bash
/mnt/data/
├── media/              <-- The "One Folder to Rule Them All"
│   ├── movies/         # qBittorrent downloads here -> Jellyfin plays from here
│   └── shows/
└── nextcloud/          # Private Cloud Data


Here is the Markdown content starting from the **Unified Storage / Docker Volume** section to the end of the file.

```markdown
## ⚙️ The "Unified Storage" Pattern

We avoided duplicate files and permission errors by pointing both the Downloader (qBittorrent) and the Player (Jellyfin) to the **exact same physical directory**.

**The Directory Structure:**
```bash
/mnt/data/
├── media/              <-- The "One Folder to Rule Them All"
│   ├── movies/         # qBittorrent downloads here -> Jellyfin plays from here
│   └── shows/
└── nextcloud/          # Private Cloud Data

```

**Docker Volume Mapping:**

* **qBittorrent:** `/mnt/data/media:/downloads`
* **Jellyfin:** `/mnt/data/media:/media`
* *Result:* As soon as a download finishes, it instantly appears in Jellyfin without needing to be moved or copied.

---

##📦 The Software Stack (Verified)All services were containerized using Docker. Below is the verified state of the stack at the time of decommissioning (July 2025).

| Category | Service | Container Name | Purpose | Status (Final Audit) |
| --- | --- | --- | --- | --- |
| **Media** | **Jellyfin** | `jellyfin` | Media Streaming Server | 🟢 Online |
| **Downloads** | **qBittorrent** | `qbittorrent` | Automated Torrent Client | 🟢 Up 9 Days |
| **Management** | **Bazarr** | `bazarr` | Subtitle Synchronization | 🟢 Up 9 Days |
| **Cloud** | **Nextcloud** | `nextcloud-project_app` | File Sync & Share | 🟢 Up 9 Days |
| **Database** | **MySQL/Redis** | `nextcloud-db` | Backend for Nextcloud | 🟢 Up 9 Days |
| **Security** | **Vaultwarden** | `vaultwarden` | Bitwarden Password Manager | 🟢 Up 9 Days |
| **Dashboard** | **Homepage** | `homepage` | System Dashboard | 🟢 Up 9 Days |
| **Monitoring** | **Prometheus** | `prometheus` | Metrics Collection | 🟢 Up 9 Days |
| **Monitoring** | **Grafana** | `grafana` | Data Visualization | 🔴 **Crashed (Exited 255)** |
| **Mgmt** | **Portainer** | `portainer` | Docker GUI | 🟢 Up 9 Days |

---

##🛡️ Resilience & Engineering ChallengesBuilding v1 wasn't just about installing software; it was a battle against network restrictions and resource constraints.

###🛑 Challenge 1: The "Invisible Server" (CGNAT)> **The Obstacle:** My ISP placed the network behind a Carrier-Grade NAT. The server had no public IP, making it impossible to access from outside the house.
> **The Solution:** We deployed **Cloudflare Tunnels**. By running a lightweight daemon (`cloudflared`) inside the Ubuntu VM, we established an outbound connection to Cloudflare's edge, allowing us to route traffic to `*.gharlab.com` without opening a single port on the router.

###🛑 Challenge 2: The Zombie Process Leak> **The Obstacle:** Over weeks of uptime, the server's RAM usage would creep up, and performance would degrade, requiring manual reboots.
> **The Diagnosis:** A final audit via `top` revealed **28 Zombie Processes**. The Docker monitoring stack was creating child processes that never exited cleanly, slowly consuming the PID table. This taught me the importance of PID limits in Docker Compose.

###🛑 Challenge 3: VPN Handshake Failures> **The Obstacle:** Initial attempts to use raw WireGuard failed because the handshake packets were being dropped by the ISP's aggressive firewall settings.
> **The Solution:** We pivoted to **Tailscale**. Its ability to use DERP (Designated Encrypted Relay for Packets) servers meant that even if a direct P2P connection failed, the traffic would still route securely through a relay, ensuring 100% uptime for admin access.

---

##📸 Post-Mortem & Final Audit**Date:** July 24, 2025

**Evidence:** Terminal Screenshots (Archived in `/docs/images`)

Before decommissioning the VM, I performed a final system audit. The state of the machine perfectly summarized why a migration was necessary.

###1. System Health (Stress Test)* **Load Average:** `0.74` (Moderate load for a dual-core VM).
* **Memory Pressure:** `39%` of 4GB used. The "Zombie Processes" were evident in the process tree, indicating a slow resource leak that would eventually crash the system.
* ![Terminal showing 28 zombie processes and memory usage](./assets/screenshots/terminal-zombie-processes.png)

###2. Service Fragility* **Grafana Crash:** As seen in the logs, the `grafana` container had exited with **Code 255**. This proved that running a heavy JVM/Go-based observability stack on a resource-constrained VirtualBox VM was too fragile for production use.
![Docker PS command showing Grafana crash](./assets/screenshots/docker-ps-output.png)

###3. The "Single Point of Failure"* The most critical flaw was architectural. The entire lab lived on a single 500GB virtual disk (`.vdi`). There was no RAID, no ZFS bit-rot protection, and no snapshots. One corruption event would have wiped 1TB of data.

> **Final Verdict:** Ghar Labs v1 was a successful Proof of Concept, but it lacked the data integrity required for a permanent home server. This necessity drove the migration to **TrueNAS Scale (v2)**.

---

*Maintained by Pratyush | 2024-2025*

```

```

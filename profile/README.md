<p align="center">
  <img src="https://avatars.githubusercontent.com/u/257679218" alt="CubeOS Logo" width="120">
</p>
<h1 align="center">CubeOS</h1>
<p align="center">
  <strong>What Synology DSM is for NAS, CubeOS is for Raspberry Pi</strong>
</p>
<p align="center">
  An open-source ARM64 server OS with web management, offline-first design, and expedition-grade hardware support
</p>
<p align="center">
  <a href="https://cubeos.app">Website</a> •
  <a href="https://github.com/cubeos-app/docs">Documentation</a> •
  <a href="https://github.com/cubeos-app/releases">Download</a> •
  <a href="https://github.com/orgs/cubeos-app/discussions">Community</a>
</p>

---

## What is CubeOS?

**CubeOS** transforms a Raspberry Pi into a self-contained server with a consumer-friendly web interface. Unlike similar projects that focus solely on home servers, CubeOS is designed for **offline-first operation** — making it ideal for field deployments, expeditions, emergency response, and air-gapped environments.

### Why CubeOS?

| Need | CubeOS Solution |
|------|-----------------|
| Run Docker apps without internet | Local registry with 20+ pre-cached apps |
| Manage services without SSH | Web dashboard with app store |
| Create a portable network | Built-in WiFi access point with DHCP/DNS |
| Survive network changes | Three network modes: offline, ethernet uplink, WiFi uplink |
| Access hardware from containers | Hardware Abstraction Layer (HAL) with REST API |
| Prepare for multi-node clusters | Docker Swarm architecture (single-node today, cluster-ready) |

---

## Key Features

### Web Dashboard
Manage Docker containers, view logs, monitor CPU/RAM/storage, and install apps — all from your browser. No terminal required.

### WiFi Access Point
Turn your Pi into a portable hotspot. Three network modes adapt to any environment:
- **OFFLINE** — Air-gapped access point, no internet required
- **ONLINE_ETH** — Access point with internet via Ethernet
- **ONLINE_WIFI** — Access point with internet via USB WiFi dongle

### One-Click App Store
Install self-hosted apps instantly. CasaOS-compatible app manifests with automatic ARM64 architecture checking.

### Hardware Abstraction Layer (HAL)
A unique REST API providing containerized access to Pi hardware without requiring privileged containers:

| Category | Capabilities |
|----------|-------------|
| **System** | Temperature, throttling, uptime, boot config |
| **Power** | UPS HATs, battery status, RTC, wake alarms |
| **Storage** | NVMe, USB drives, SMART health, auto-mount |
| **Network** | Interface stats, AP clients, traffic monitoring |
| **GPIO/I2C** | Pin control, bus scanning, sensor reading |
| **Audio** | ALSA devices, volume control, capture |
| **Location** | GPS/GNSS receivers, position tracking |
| **Communication** | Cellular modems, Meshtastic LoRa, Iridium satellite |

### Offline-First Architecture
- Local Docker registry caches images for offline installation
- Pi-hole provides DNS/DHCP without upstream connectivity  
- All core functionality works in airplane mode

### VPN & Privacy
- WireGuard and OpenVPN client support
- Per-app Tor routing for selective anonymity
- Firewall management with NAT control

### Storage Integration
- Mount SMB/NFS shares as app volumes
- USB drive auto-detection and mounting
- Backup to local, network, or external storage

---

## Architecture

CubeOS uses a **Docker Swarm + HAL** architecture that separates hardware access from application logic:

```
┌─────────────────────────────────────────────────────────────────┐
│  Dashboard (Vue.js)          ← Web UI for users                 │
└──────────────────────────────────┬──────────────────────────────┘
                                   │ REST API
┌──────────────────────────────────▼──────────────────────────────┐
│  CubeOS API (Go)             ← Auth, business logic, Swarm mgmt │
└──────────────────────────────────┬──────────────────────────────┘
                                   │ Internal API
┌──────────────────────────────────▼──────────────────────────────┐
│  HAL Service                 ← Hardware access (privileged)     │
└──────────────────────────────────┬──────────────────────────────┘
                                   │
┌──────────────────────────────────▼──────────────────────────────┐
│  Raspberry Pi Hardware       ← GPIO, I2C, USB, Network, etc.    │
└─────────────────────────────────────────────────────────────────┘
```

**Why this matters:**
- Application containers run unprivileged (better security)
- Hardware changes don't break apps (HAL abstracts the differences)
- Swarm provides self-healing, rolling updates, and future multi-node support
- Only 3 services need host access (Pi-hole, NPM, HAL) — everything else is isolated

---

## Target Users

- **Self-hosting enthusiasts** — Run your own cloud services at home
- **Field deployments** — Remote monitoring, data collection, edge computing
- **Expeditions** — Offline operation with GPS, satellite, and mesh networking
- **Emergency response** — Portable infrastructure that works without internet
- **IoT/Edge computing** — Sensor integration via HAL, containerized processing
- **Education** — Learn Docker, networking, and Linux on real hardware

---

## Repositories

| Repo | Description |
|------|-------------|
| [**api**](https://github.com/cubeos-app/api) | Go backend — REST API, Docker Swarm orchestration, HAL client |
| [**dashboard**](https://github.com/cubeos-app/dashboard) | Vue.js 3 frontend — Web management interface |
| [**coreapps**](https://github.com/cubeos-app/coreapps) | Docker configs for system services (HAL, Pi-hole, NPM, etc.) |
| [**scripts**](https://github.com/cubeos-app/scripts) | Deployment, backup, and utility scripts |
| [**releases**](https://github.com/cubeos-app/releases) | Packer configs and SD card images |
| [**docs**](https://github.com/cubeos-app/docs) | User guides, API reference, architecture docs |

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Backend** | Go 1.24+, chi router, SQLite (pure Go) |
| **Frontend** | Vue.js 3, Pinia, Tailwind CSS |
| **Orchestration** | Docker Swarm (single-node, multi-node ready) |
| **HAL** | Go service with 80+ REST endpoints, Swagger UI |
| **Networking** | Pi-hole (DNS/DHCP), Nginx Proxy Manager, hostapd |
| **Registry** | Local Docker registry with pull-through cache |
| **Target** | Raspberry Pi 5/4, any ARM64 SBC (2GB+ RAM) |

---

## Quick Start

1. **Download** the latest image from [Releases](https://github.com/cubeos-app/releases)
2. **Flash** to SD card using [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
3. **Boot** your Pi and connect to the `CubeOS-Setup` WiFi network
4. **Open** `http://cubeos.cube` in your browser
5. **Complete** the setup wizard

Default credentials: `admin` / `admin` (change on first login)

---

## Network Configuration

| Setting | Value |
|---------|-------|
| Subnet | `10.42.24.0/24` |
| Gateway | `10.42.24.1` |
| Domain | `cubeos.cube` |
| DHCP Range | `10.42.24.10 - 10.42.24.250` |

### Port Allocation

| Range | Purpose |
|-------|---------|
| 6000-6009 | Infrastructure (Pi-hole, NPM) |
| 6010-6019 | Platform (API, Dashboard, Dozzle) |
| 6020-6029 | VPN (WireGuard, OpenVPN, Tor) |
| 6030-6039 | AI/ML (Ollama, ChromaDB) |
| 6100-6999 | User applications |

---

## How CubeOS Compares

| Feature | CubeOS | CasaOS | Umbrel | Home Assistant OS |
|---------|--------|--------|--------|-------------------|
| Orchestration | Docker Swarm | Docker Compose | Docker Compose | Supervisor (custom) |
| Offline-first | Yes | No | No | No |
| HAL for hardware | 80+ endpoints | Direct access | N/A | Add-on config |
| Multi-node ready | Swarm native | No | No | No |
| WiFi AP built-in | Three modes | No | No | No |
| Expedition hardware | GPS, satellite, mesh | No | No | Via integrations |
| App ecosystem | CasaOS-compatible | Native | Native | Add-ons |

---

## Contributing

We welcome contributions! See our [Contributing Guide](https://github.com/cubeos-app/docs/blob/main/CONTRIBUTING.md).

- **Report bugs** via [GitHub Issues](https://github.com/cubeos-app/api/issues)
- **Discuss ideas** in [GitHub Discussions](https://github.com/orgs/cubeos-app/discussions)
- **Submit PRs** to any repository

### Development Setup

```bash
# Clone repositories
git clone https://github.com/cubeos-app/api.git
git clone https://github.com/cubeos-app/dashboard.git
git clone https://github.com/cubeos-app/coreapps.git

# API development
cd api && go build ./cmd/cubeos-api/

# Dashboard development  
cd dashboard && npm install && npm run dev
```

---

## Roadmap

- [x] Docker Swarm orchestration
- [x] HAL with 80+ hardware endpoints
- [x] CasaOS app compatibility
- [x] WiFi access point with three modes
- [ ] Raspberry Pi Imager listing
- [ ] Multi-node Swarm clustering
- [ ] Backup/restore system
- [ ] First-boot setup wizard
- [ ] Read-only root filesystem option

---

## License

CubeOS is open source under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0).

Commercial derivatives and white-label products are permitted — see LICENSE for details.

---

<p align="center">
  <sub>Built for expeditions, emergencies, and everyone who wants their own cloud</sub>
</p>

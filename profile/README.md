<p align="center">
  <img src="https://avatars.githubusercontent.com/u/257679218" alt="CubeOS Logo" width="140">
</p>
<h1 align="center">CubeOS</h1>
<p align="center">
  <strong>The open-source server OS for Raspberry Pi and ARM64 edge devices</strong>
</p>
<p align="center">
  <em>What Synology DSM is for NAS, CubeOS is for Raspberry Pi — with offline-first design,<br>self-healing containers, and expedition-grade hardware support.</em>
</p>
<p align="center">
  <a href="https://github.com/cubeos-app/api"><img src="https://img.shields.io/badge/backend-Go_1.24-00ADD8?logo=go&logoColor=white" alt="Go"></a>
  <a href="https://github.com/cubeos-app/dashboard"><img src="https://img.shields.io/badge/frontend-Vue.js_3-4FC08D?logo=vuedotjs&logoColor=white" alt="Vue.js"></a>
  <a href="https://www.apache.org/licenses/LICENSE-2.0"><img src="https://img.shields.io/badge/license-Apache_2.0-blue" alt="License"></a>
  <a href="https://github.com/orgs/cubeos-app/discussions"><img src="https://img.shields.io/badge/community-discussions-orange?logo=github" alt="Discussions"></a>
</p>
<p align="center">
  <a href="https://cubeos.app">Website</a> ·
  <a href="https://github.com/cubeos-app/docs">Documentation</a> ·
  <a href="https://github.com/cubeos-app/releases">Download</a> ·
  <a href="https://github.com/orgs/cubeos-app/discussions">Community</a>
</p>

---

CubeOS transforms a Raspberry Pi into a self-contained server with a consumer-friendly web dashboard, one-click app store, and a unique Hardware Abstraction Layer. Unlike other platforms that focus solely on home servers, CubeOS is built for **offline-first operation** — making it equally at home on your desk, in a remote field station, or aboard an expedition vehicle crossing the Sahara.

**Docker Swarm** provides self-healing containers with automatic reconciliation. If a service crashes at 3 AM, Swarm restarts it — no human intervention, no SSH, no scripts. Combined with a local Docker registry that caches images for offline installation, CubeOS runs reliably in environments where "just pull the latest image" isn't an option.

<p align="center">
  <img src="https://raw.githubusercontent.com/cubeos-app/docs/main/assets/dashboard-preview.png" alt="CubeOS Dashboard" width="720">
</p>

---

## Why CubeOS Exists

The Raspberry Pi and ARM64 SBC ecosystem has lacked a serious, open-source server operating system — one that provides web management, container orchestration, and hardware integration without requiring a computer science degree to operate.

Projects like CasaOS showed the world that a Raspberry Pi could be a personal server. But when its parent company shifted focus to proprietary, x86-only commercial products, the ARM and Raspberry Pi community was left without an actively maintained platform. Thousands of self-hosters on Raspberry Pi hardware found themselves on an abandoned codebase with 779 unresolved issues, no stable releases, and no upgrade path.

**CubeOS picks up where that vision left off — and goes further.**

Where previous platforms used Docker Compose with no self-healing, CubeOS uses **Docker Swarm** with automatic service reconciliation. Where others required internet connectivity, CubeOS works in **airplane mode**. Where others ignored the hardware, CubeOS exposes **80+ REST endpoints** through a Hardware Abstraction Layer that lets any container read GPS coordinates, query battery status, or send satellite messages — without needing privileged access.

And the commitment is carved into the license: **Apache 2.0, forever.** No bait-and-switch. No paywall. No "community edition" with artificial limits.

> *CubeOS runs 800+ apps from the CasaOS ecosystem through its Compose-to-Swarm compatibility layer — giving you the same app store experience, with self-healing containers underneath.*

---

## Key Features

### Web Dashboard

Manage Docker containers, view real-time logs, monitor CPU/RAM/storage/temperature, and install apps — all from your phone or laptop browser. No terminal required. Responsive design works on any screen size.

### Self-Healing Containers

Docker Swarm continuously monitors every service. If a container crashes, Swarm automatically restarts it. If a node goes down in a multi-node cluster, workloads redistribute. This is the same orchestration technology used in production data centres — running on a $80 Raspberry Pi.

### WiFi Access Point

CubeOS turns your Pi into a portable network with integrated DNS and DHCP. Three modes adapt to any environment:

| Mode | Behaviour |
|------|-----------|
| **OFFLINE** | Air-gapped access point — no internet required |
| **ONLINE_ETH** | Access point with internet via Ethernet uplink |
| **ONLINE_WIFI** | Access point with internet via USB WiFi dongle |

Connect your phone, laptop, or IoT devices to the CubeOS network and access the dashboard at `http://cubeos.cube`.

### One-Click App Store

Install self-hosted apps instantly. CubeOS is compatible with the **CasaOS app ecosystem** — over 800 apps across 15+ community-maintained stores including BigBearCasaOS (200+ apps), LinuxServer AppStore (100+ apps), and more. Apps are automatically checked for ARM64 architecture compatibility.

### Hardware Abstraction Layer (HAL)

A unique privileged microservice that exposes Pi hardware through a REST API. Application containers stay unprivileged and secure — they talk to the HAL, not to raw `/dev` devices.

| Category | Capabilities |
|----------|-------------|
| **System** | Temperature, throttling, uptime, boot config, kernel info |
| **Power** | UPS HATs, battery level, RTC, wake alarms, safe shutdown |
| **Storage** | NVMe, USB drives, SMART health, auto-mount, partition info |
| **Network** | Interface stats, AP clients, traffic monitoring, WiFi scanning |
| **GPIO / I2C** | Pin control, bus scanning, sensor reading, PWM |
| **Audio** | ALSA devices, volume control, capture, playback |
| **Location** | GPS/GNSS receivers, position tracking, fix quality |
| **Communication** | Cellular modems, Meshtastic LoRa, Iridium satellite |

### Offline-First Architecture

Every component is designed to work without internet connectivity:

- **Local Docker registry** caches images for offline app installation
- **Pi-hole** provides DNS/DHCP without upstream connectivity
- **All core services** function in airplane mode
- **Local AI** via Ollama with ChromaDB RAG — no cloud API keys needed

### VPN & Privacy

- WireGuard and OpenVPN client/server support
- Per-app Tor routing for selective anonymity
- Firewall management with NAT control

### Storage Integration

- Mount SMB/NFS shares as app volumes
- USB drive auto-detection and mounting
- Backup to local, network, or external storage

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│  Browser / Phone                                                    │
└────────────────────────────────┬────────────────────────────────────┘
                                 │ HTTPS (Nginx Proxy Manager)
┌────────────────────────────────▼────────────────────────────────────┐
│  Dashboard (Vue.js 3)          — Web UI, responsive, theme-aware    │
└────────────────────────────────┬────────────────────────────────────┘
                                 │ REST API
┌────────────────────────────────▼────────────────────────────────────┐
│  CubeOS API (Go)               — Auth, Swarm orchestration, apps    │
└──────────────┬─────────────────┬────────────────────────────────────┘
               │                 │ Internal API
               │  ┌──────────────▼──────────────────────────────────┐
               │  │  HAL Service (Go, privileged)                    │
               │  │  80+ REST endpoints · Swagger UI · SSE streams   │
               │  └──────────────┬──────────────────────────────────┘
               │                 │
┌──────────────▼─────────────────▼────────────────────────────────────┐
│  Raspberry Pi / ARM64 SBC                                           │
│  GPIO · I2C · USB · Serial · WiFi · Bluetooth · NVMe · GPS         │
└─────────────────────────────────────────────────────────────────────┘
```

**Design principles:**

- **Unprivileged by default** — Only 3 services need host access (Pi-hole, NPM, HAL). Everything else runs in isolated Swarm stacks.
- **HAL abstracts hardware** — Apps don't care if the GPS is USB, I2C, or Bluetooth. They call the same REST endpoint.
- **Swarm enables resilience** — Self-healing, rolling updates, secrets management, and future multi-node clustering.
- **Backend-for-Frontend pattern** — The Dashboard talks only to the API. The API talks to HAL. Security boundaries are enforced at every layer.

---

## How CubeOS Compares

| Feature | CubeOS | CasaOS | Umbrel | Home Assistant OS |
|---------|--------|--------|--------|-------------------|
| **Status** | Active development | Abandoned (last release Dec 2024) | Active | Active |
| **License** | Apache 2.0 | Apache 2.0 | PolyForm Noncommercial | Apache 2.0 |
| **Orchestration** | Docker Swarm (self-healing) | Docker Compose | Docker Compose | Custom Supervisor |
| **Offline operation** | Full (local registry + AP) | No | No | Partial |
| **Hardware abstraction** | 80+ REST endpoints | None | None | Add-on config files |
| **Multi-node ready** | Swarm native | No | No | No |
| **WiFi access point** | 3 modes built-in | No | No | No |
| **ARM64 native** | Pi 5/4, any ARM64 SBC | ARM64 + x86 | Pi 5, x86 | Pi 5/4 only |
| **Expedition hardware** | GPS, satellite, LoRa mesh | No | No | Via integrations |
| **App ecosystem** | 800+ (CasaOS-compatible) | 800+ (native) | ~200 | ~3000 (add-ons) |
| **Paywall** | None, ever | N/A (abandoned) | Hardware-gated features | None |

---

## The CubeOS Platform

CubeOS is not a single product — it's a **platform** that adapts to different hardware and use cases through specialised editions. The core operating system remains the same; each edition adds configuration profiles, HAL drivers, and pre-configured app stacks for its target environment.

<p align="center">

```
                              ┌──────────────┐
                              │   CubeOS     │
                              │    Core      │
                              │              │
                              │  Dashboard   │
                              │  API + HAL   │
                              │  Swarm       │
                              │  App Store   │
                              └──────┬───────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
     ┌────────▼────────┐   ┌────────▼────────┐   ┌────────▼────────┐
     │  CubeOS-NAS     │   │ CubeOS-MeshSat  │   │  CubeOS-Edge   │
     │                 │   │                  │   │                 │
     │  RAID / ZFS     │   │  LoRa Mesh       │   │  NVIDIA DGX    │
     │  Encrypted      │   │  Iridium Sat     │   │  GPU / AI      │
     │  File Sharing   │   │  SOS Beacon      │   │  K3s Clusters  │
     │  Backup         │   │  Offline Comms   │   │  Verticals     │
     └─────────────────┘   └──────────────────┘   └────────────────┘
```

</p>

---

### CubeOS-NAS — Network-Attached Storage Edition

> *Turn a Raspberry Pi 5 with an NVMe HAT into a proper NAS — with a web UI, RAID, encrypted folders, and Samba/NFS sharing.*

CubeOS-NAS extends the core platform with storage management features that compete with dedicated NAS operating systems, while retaining CubeOS's unique advantages: offline operation, self-healing containers, and a $0 price tag with no artificial limits.

**Planned capabilities:**

| Feature | Description |
|---------|-------------|
| **RAID management** | GUI for mdadm RAID 0/1/5/6 — create, monitor, rebuild arrays |
| **Encrypted volumes** | LUKS or gocryptfs encrypted folders with web-based unlock |
| **File sharing** | Samba (SMB) and NFS server with user/group permission management |
| **Backup engine** | Scheduled backups to local, USB, network, or cloud (S3-compatible) targets |
| **Storage dashboard** | Visual disk health (SMART), usage graphs, temperature monitoring |
| **External drives** | Auto-mount USB/NVMe with configurable mount points |
| **Time Machine** | Apple Time Machine target support via Samba |

**Target hardware:** Raspberry Pi 5 with NVMe HAT (Pimoroni, Geekworm, etc.), USB SATA adapters, or any ARM64 SBC with multiple storage interfaces. Supports 2GB+ RAM.

**Status:** Planning phase. RAID management and encrypted volumes are the priority features for the first release.

---

### CubeOS-MeshSat — Mesh Network + Satellite Bridge

> *Bridge an entire LoRa mesh network to the Iridium satellite constellation. Send texts, GPS positions, and SOS alerts from anywhere on Earth — no cell towers required.*

CubeOS-MeshSat turns a CubeOS node into a **Meshtastic-to-Iridium satellite gateway**. One Raspberry Pi with a Heltec LoRa radio and a RockBLOCK 9603 satellite modem becomes the bridge between a local mesh network and global satellite coverage.

**Core data flow:**

```
Mesh Node ──LoRa──▶ Heltec Radio ──USB/BLE──▶ CubeOS HAL ──▶ cubeos-meshsat
                                                                    │
                                              CubeOS HAL ◀── RockBLOCK 9603
                                                                    │
                                                           Iridium Satellite
                                                                    │
                                                       Ground Control Webhook
```

**Planned capabilities:**

| Feature | Description |
|---------|-------------|
| **Mesh-to-satellite bridge** | Forward text messages and GPS positions from any Meshtastic node to Iridium |
| **Bidirectional relay** | Receive satellite messages (MT) and broadcast them to the mesh network |
| **SOS beacon** | One-button emergency alert with GPS coordinates, sent via satellite with delivery confirmation |
| **Dual radio transport** | USB serial (Heltec LoRa V4) and Bluetooth LE (LilyGo T-Echo) with automatic failover |
| **Message queue** | Persistent SQLite queue with automatic retry and exponential backoff |
| **Mesh node map** | Web dashboard showing all mesh nodes, positions, battery levels, and signal quality |
| **Ground Control integration** | Webhook-based relay for MO/MT messages via RockBLOCK Web Services |

**Hardware requirements:**

| Component | Purpose | Connection |
|-----------|---------|------------|
| Raspberry Pi 5/4 | Gateway computer | — |
| Heltec LoRa V4 (ESP32-S3) | Meshtastic radio, 868/915 MHz | USB serial |
| *or* LilyGo T-Echo (nRF52840) | Meshtastic radio (BLE option) | Bluetooth LE |
| RockBLOCK 9603 | Iridium SBD satellite modem | USB via FTDI |
| GPS antenna | Satellite fix + mesh positioning | Built into LoRa radio |
| Iridium antenna | Satellite uplink (clear sky view) | SMA on RockBLOCK |

**How it works:** The HAL provides two new driver modules — `meshtastic` (LoRa mesh communication) and `iridium` (satellite SBD protocol). The `cubeos-meshsat` app runs as a standard unprivileged Docker container, consuming real-time mesh messages via SSE from the HAL and forwarding them through the Iridium modem. Messages are compressed into a custom binary format to fit within Iridium's 340-byte MO limit. The SOS system implements a full state machine (IDLE → SENDING → WAIT_ACK → CONFIRMED) with GPIO trigger support for physical panic buttons.

**Target users:** Expeditions, maritime operations, remote field research, search and rescue, disaster response, off-grid communities.

**Status:** Architecture complete. HAL driver specifications defined for both Meshtastic (USB + BLE transports) and Iridium (AT command protocol). Implementation follows a 5-phase plan: HAL Iridium driver → HAL Meshtastic driver (USB) → HAL Meshtastic driver (BLE) → Bridge core → Web UI.

---

### CubeOS-Edge — ARM64 AI Appliance Manager

> *The missing management plane for NVIDIA DGX Spark and ARM64 edge AI devices. Transform a €3,000 desktop AI supercomputer into a business-ready appliance — with a web console, vertical solution templates, and one-click deployment.*

NVIDIA ships the DGX Spark (128 GB unified GPU memory, ARM64) with zero web-based management. No browser dashboard, no container management UI, no out-of-band recovery, no fleet orchestration. The only interfaces are SSH and a local-only GPU telemetry page. CubeOS-Edge fills this gap.

**Planned capabilities:**

| Feature | Description |
|---------|-------------|
| **Web console** | Full container lifecycle management with GPU monitoring (nvidia-smi), system health, storage, and user management on port 6010 |
| **Setup wizard** | First-boot experience: "What is your use case?" → deploys a pre-configured Docker stack for the selected vertical |
| **Vertical templates** | One-click deployment for Legal/Compliance, Agriculture, Healthcare, or custom AI pipelines |
| **GPU dashboard** | Real-time VRAM usage, inference throughput, temperature, and utilisation graphs |
| **K3s orchestration** | Kubernetes-based clustering that supports both load-balanced replicas and distributed inference across multiple DGX Spark units via 200 GbE ConnectX-7 |
| **Vertical WebUIs** | Separate customer-facing application per vertical — built for business users, not IT administrators |

**Scaling model:**

| Configuration | Capability |
|---------------|------------|
| **1 × DGX Spark** | Full AI pipeline: 70B LLM (40 GB) + embeddings (3 GB) + vector DB + services. Serves 25 concurrent users. |
| **2 × DGX Spark** | Load-balanced replicas — doubles throughput, adds redundancy and zero-downtime updates |
| **4 × DGX Spark** | Distributed inference — pool GPU memory via Ray cluster for 200B+ parameter models |

**Architecture difference:** CubeOS-Edge uses **K3s** instead of Docker Swarm for orchestration. While Swarm is ideal for single-node Raspberry Pi deployments (lightweight, simple, Docker-native), DGX Spark clusters require Kubernetes-level features: Layer 7 routing, GPU scheduling, distributed inference coordination, and multi-node storage. The CubeOS API backend and Vue.js dashboard remain the same — only the orchestration layer changes.

**Target hardware:** NVIDIA DGX Spark, MSI EdgeXpert, Jetson AGX Orin, any ARM64 Linux device with NVIDIA GPU. Also applicable to future GB-series products and third-party OEM variants.

**Business model:** The management platform is open-source (Apache 2.0). Revenue comes from vertical solution templates, enterprise support contracts, and custom WebUI development.

**Status:** Executive summary and architecture complete. First target vertical: Greek regulatory compliance monitoring (FEK corpus, 1M documents, 70B LLM). Fork-ready from CubeOS core — requires stripping Pi-specific components and adapting the HAL for GPU hardware.

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Backend** | Go 1.24+, chi router, SQLite (pure Go, no CGO) |
| **Frontend** | Vue.js 3 + Composition API, Pinia, Tailwind CSS |
| **Orchestration** | Docker Swarm (single-node, multi-node ready) |
| **HAL** | Go privileged service, 80+ REST endpoints, Swagger UI |
| **DNS / DHCP** | Pi-hole v6 |
| **Reverse proxy** | Nginx Proxy Manager |
| **Registry** | Local Docker registry with pull-through cache |
| **AI** | Ollama + ChromaDB (RAG) |
| **Monitoring** | Dozzle (container logs) |
| **Target hardware** | Raspberry Pi 5/4, any ARM64 SBC (2GB+ RAM) |

---

## Repositories

| Repo | Description |
|------|-------------|
| [**api**](https://github.com/cubeos-app/api) | Go backend — REST API, Docker Swarm orchestration, HAL client, 119+ handlers |
| [**dashboard**](https://github.com/cubeos-app/dashboard) | Vue.js 3 frontend — Web management interface with Pinia state management |
| [**coreapps**](https://github.com/cubeos-app/coreapps) | Docker Compose/Swarm configs for system services (HAL, Pi-hole, NPM, Ollama, etc.) |
| [**scripts**](https://github.com/cubeos-app/scripts) | Deployment, backup, watchdog, and utility scripts |
| [**releases**](https://github.com/cubeos-app/releases) | Packer image builder and Raspberry Pi Imager manifest |
| [**docs**](https://github.com/cubeos-app/docs) | Architecture documentation, user guides, API reference |

---

## Quick Start

**1.** Download the latest image from [Releases](https://github.com/cubeos-app/releases)

**2.** Flash to SD card using [Raspberry Pi Imager](https://www.raspberrypi.com/software/)

**3.** Boot your Pi and connect to the **`CubeOS-Setup`** WiFi network

**4.** Open **`http://cubeos.cube`** in your browser

**5.** Complete the setup wizard

Default credentials: `admin` / `admin` (change on first login)

### Network Defaults

| Setting | Value |
|---------|-------|
| Subnet | `10.42.24.0/24` |
| Gateway / Dashboard | `10.42.24.1` |
| Domain | `cubeos.cube` |
| DHCP range | `10.42.24.10 – 10.42.24.250` |

### Port Allocation

| Range | Purpose |
|-------|---------|
| `6000–6009` | Infrastructure (Pi-hole, NPM) |
| `6010–6019` | Platform (API, Dashboard, Dozzle) |
| `6020–6029` | VPN (WireGuard, OpenVPN, Tor) |
| `6030–6039` | AI/ML (Ollama, ChromaDB) |
| `6100–6999` | User applications |

---

## Development

```bash
# Clone the core repositories
git clone https://github.com/cubeos-app/api.git
git clone https://github.com/cubeos-app/dashboard.git
git clone https://github.com/cubeos-app/coreapps.git

# Backend development (Go)
cd api && go build ./cmd/cubeos-api/

# Frontend development (Vue.js)
cd dashboard && npm install && npm run dev
```

All repositories have GitLab CI/CD pipelines that auto-deploy to the Pi on push to `main`. The development workflow is laptop → GitLab → Pi — no manual intervention on the device.

---

## Roadmap

### Core Platform

- [x] Docker Swarm orchestration with self-healing
- [x] HAL with 80+ hardware REST endpoints
- [x] CasaOS app store compatibility (800+ apps)
- [x] WiFi access point with three network modes
- [x] Local AI with Ollama + ChromaDB RAG
- [x] GitLab CI/CD auto-deployment pipelines
- [x] Complete Swagger/OpenAPI documentation
- [ ] Raspberry Pi Imager official listing
- [ ] First-boot setup wizard
- [ ] Multi-node Swarm clustering
- [ ] Backup and restore system
- [ ] Read-only root filesystem option

### CubeOS-NAS

- [ ] RAID management GUI (mdadm)
- [ ] Encrypted volumes (LUKS / gocryptfs)
- [ ] Samba/NFS file sharing with user management
- [ ] Scheduled backup engine
- [ ] Time Machine target support

### CubeOS-MeshSat

- [ ] HAL Iridium driver (RockBLOCK AT commands, SBD protocol)
- [ ] HAL Meshtastic driver — USB serial transport
- [ ] HAL Meshtastic driver — BLE transport
- [ ] Mesh-to-satellite bridge core with message queue
- [ ] SOS beacon with state machine and GPIO trigger
- [ ] Web UI (mesh node map, message history, diagnostics)

### CubeOS-Edge

- [ ] Fork core platform for DGX Spark / ARM64 GPU hardware
- [ ] K3s orchestration layer (replacing Swarm)
- [ ] nvidia-smi GPU monitoring integration
- [ ] Setup wizard with vertical selection
- [ ] First vertical template: Legal/Compliance AI pipeline

---

## Contributing

We welcome contributions of all kinds — code, documentation, bug reports, feature ideas, and app store packages.

- **Report bugs** → [GitHub Issues](https://github.com/cubeos-app/api/issues)
- **Discuss ideas** → [GitHub Discussions](https://github.com/orgs/cubeos-app/discussions)
- **Submit PRs** → Any repository. See our [Contributing Guide](https://github.com/cubeos-app/docs/blob/main/CONTRIBUTING.md).

Whether you're running CubeOS on a Pi at home, deploying it on a research vessel, or building AI appliances for enterprise customers — you're part of the same community.

---

## License

CubeOS is open source under the **[Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0)**.

You can use, modify, distribute, and build commercial products on CubeOS — with no royalties, no paywall, and no artificial feature limits. This is a permanent commitment, not a growth strategy.

---

<p align="center">
  <strong>Built for expeditions, emergencies, and everyone who wants their own cloud.</strong>
  <br><br>
  <sub>CubeOS · Apache 2.0 · Forever free · Forever open</sub>
</p>

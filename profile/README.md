<p align="center">
  <img src="https://avatars.githubusercontent.com/u/257679218" alt="CubeOS Logo" width="140">
</p>
<h1 align="center">CubeOS</h1>
<p align="center">
  <strong>The open-source server OS for Raspberry Pi and ARM64 edge devices</strong>
</p>
<p align="center">
  <em>What Synology DSM is for NAS, CubeOS is for Raspberry Pi — with offline-first design,<br>self-healing containers, and network-transparent hardware access across every node.</em>
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

CubeOS transforms a Raspberry Pi into a self-contained server with a consumer-friendly web dashboard, one-click app store, and a unique **Hardware Abstraction Layer** (HAL). Plug a GPS receiver into Node 1 and a LoRa radio into Node 2 — any container on any node can access both devices through a unified REST API, as if the hardware were local. CubeOS makes hardware location irrelevant.

Built for **offline-first operation**, CubeOS is equally at home on your desk, in a remote field station, or aboard an expedition vehicle crossing the Sahara. **Docker Swarm** provides self-healing containers with automatic reconciliation — if a service crashes at 3 AM, Swarm restarts it without human intervention. A local Docker registry caches images for offline installation, so CubeOS runs reliably where "just pull the latest image" isn't an option.

<p align="center">
  <img src="https://raw.githubusercontent.com/cubeos-app/docs/main/assets/dashboard-preview.png" alt="CubeOS Dashboard" width="720">
</p>

---

## Why CubeOS Exists

Docker Swarm gives you self-healing containers, rolling updates, secrets management, and automatic workload scheduling. But it's **blind to hardware**. It can schedule based on CPU and RAM, but it has no idea that Node 1 has a Coral TPU plugged in, Node 2 has a Zigbee coordinator, and Node 3 has a GPS receiver. You can set node labels manually — but if someone moves the Coral from Node 1 to Node 3, the labels are wrong and your service deploys to the wrong place.

CubeOS was born from this exact frustration. Docker Swarm had the orchestration features we needed. It just couldn't see the hardware underneath it. So we built the missing layer: a **Go daemon on each node** that auto-discovers all connected hardware, dynamically manages Swarm node labels via the Docker API, and exposes every device through REST endpoints that any container on any node can reach over the network.

**The result:** Docker Swarm handles container lifecycle. The HAL gives Swarm eyes. Containers follow hardware automatically. And low-bandwidth devices like serial radios, GPS receivers, and satellite modems become network-transparent — accessible from any node in the cluster without moving the container to the device.

No other lightweight platform does this. Kubernetes solves it with Node Feature Discovery plus a constellation of device plugins and operators — hundreds of thousands of lines across dozens of projects that only work inside Kubernetes. CubeOS provides the same capability in a single binary, running on a $80 Raspberry Pi, with zero Kubernetes overhead.

> *CubeOS runs 800+ apps from the CasaOS ecosystem through its Compose-to-Swarm compatibility layer — giving you the same app store experience, with self-healing containers underneath.*

---

## Key Features

### Web Dashboard

Manage Docker containers, view real-time logs, monitor CPU/RAM/storage/temperature, and install apps — all from your phone or laptop browser. No terminal required. Responsive design works on any screen size.

### Self-Healing Containers

Docker Swarm continuously monitors every service. If a container crashes, Swarm automatically restarts it. If a node goes down in a multi-node cluster, workloads redistribute. Combined with the HAL's dynamic node labelling, services with hardware constraints automatically follow their devices — move a Coral TPU from Node 1 to Node 3 and the dependent container migrates with it. This is production-grade orchestration running on a $80 Raspberry Pi.

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

The HAL is a privileged Go microservice running on each node that solves three problems Docker Swarm ignores:

**1. Automatic hardware discovery and Swarm label management.** The HAL scans USB, PCIe, I2C, GPIO, and system buses on startup and on hotplug events (via udev). It translates what it finds into Docker Swarm node labels through the Docker API — automatically. Plug a Coral TPU into Node 2 and the label `hw.usb.coral-tpu=true` appears. Unplug it, the label disappears. Swarm services with hardware constraints (`--constraint 'node.labels.hw.usb.coral-tpu==true'`) automatically reschedule to follow the hardware. No SSH, no manual labels, no human memory required.

**2. Network-transparent device access.** Low-bandwidth devices (serial radios, GPS receivers, sensors, satellite modems, GPIO, I2C) are exposed as REST API endpoints that any container on any node can call over the network. A Meshtastic gateway container running on Node 1 can read the GPS position from a receiver plugged into Node 3 — it's just an HTTP call to `http://node3.cubeos.cube:6005/api/v1/gps/position`. The container doesn't know or care which node the hardware is on.

**3. Semantic device APIs instead of raw passthrough.** Containers don't get raw `/dev/ttyUSB0` access. They get purpose-built endpoints: `POST /api/v1/devices/iridium/send`, `GET /api/v1/devices/meshtastic/nodes`, `POST /api/v1/gpio/write`. The Go drivers handle protocol details. Containers stay unprivileged and secure.

The HAL recognises two classes of hardware and handles them differently:

| Class | Examples | Mechanism | Container placement |
|-------|----------|-----------|-------------------|
| **Network-transparent** | Serial radios, GPS, Zigbee, LoRa, satellite modems, GPIO, I2C sensors | REST API over LAN | Runs on **any** node |
| **Co-location required** | GPU, TPU, iGPU, high-res cameras, NVMe, capture cards | Swarm labels + device passthrough | Scheduled to **device node** |

The distinction is bandwidth. A serial modem at 9600 baud generates bytes per second — trivial over a 1Gbps LAN. A Coral TPU processing video frames at 30fps requires direct PCI/USB bus access. The HAL makes this distinction automatically and applies the right strategy for each device.

**Hardware capabilities across all nodes:**

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

### Single Node

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
               │  │  Auto-discovery · Dynamic Swarm labels · udev    │
               │  └──────────────┬──────────────────────────────────┘
               │                 │
┌──────────────▼─────────────────▼────────────────────────────────────┐
│  Raspberry Pi / ARM64 SBC                                           │
│  GPIO · I2C · USB · Serial · WiFi · Bluetooth · NVMe · GPS         │
└─────────────────────────────────────────────────────────────────────┘
```

### Multi-Node Cluster

```
┌─────────────────────────────────────────────────────────────────────┐
│  Dashboard — unified view of all nodes and devices                   │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────────┐
│  CubeOS API — aggregates HAL data from every node                    │
└──────┬──────────────────┬──────────────────┬───────────────────────┘
       │                  │                  │
┌──────▼──────┐    ┌──────▼──────┐    ┌──────▼──────┐
│   Node 1    │    │   Node 2    │    │   Node 3    │
│   HAL API   │◄──►│   HAL API   │◄──►│   HAL API   │
│             │LAN │             │LAN │             │
│ 🔌 Coral TPU│    │ 🔌 GPS      │    │ 🔌 LoRa     │
│ 🔌 Zigbee   │    │ 🔌 Iridium  │    │ 🔌 NVMe     │
│             │    │             │    │             │
│  Docker     │    │  Docker     │    │  Docker     │
│  Engine     │    │  Engine     │    │  Engine     │
└─────────────┘    └─────────────┘    └─────────────┘

Any container on any node can access any device on any other node:
  GET http://node2.cubeos.cube:6005/api/v1/gps/position    ← from Node 1
  POST http://node2.cubeos.cube:6005/api/v1/iridium/send   ← from Node 3

High-bandwidth devices (GPU, TPU) use Swarm label constraints instead:
  Frigate → auto-scheduled to Node 1 (hw.usb.coral-tpu=true)
```

**Design principles:**

- **Unprivileged by default** — Only 3 services need host access (Pi-hole, NPM, HAL). Everything else runs in isolated Swarm stacks.
- **Hardware is discovered, not configured** — Plug a device in. The HAL sees it, labels the node, exposes the API. No manual setup.
- **Two strategies for two classes** — Low-bandwidth devices are network-transparent (REST API over LAN). High-bandwidth devices use co-location (Swarm labels + device passthrough).
- **Swarm stays unmodified** — No forked Docker, no custom scheduler. The HAL feeds Swarm information through standard node labels. Swarm's existing constraint-based scheduling does the rest.
- **Backend-for-Frontend pattern** — The Dashboard talks only to the API. The API talks to HAL. Security boundaries are enforced at every layer.

---

## How CubeOS Compares

| Feature | CubeOS | CasaOS | Umbrel | Home Assistant OS |
|---------|--------|--------|--------|-------------------|
| **Status** | Active development | Abandoned (last release Dec 2024) | Active | Active |
| **License** | Apache 2.0 | Apache 2.0 | PolyForm Noncommercial | Apache 2.0 |
| **Orchestration** | Docker Swarm (self-healing) | Docker Compose | Docker Compose | Custom Supervisor |
| **Hardware abstraction** | 80+ REST endpoints, auto-discovery | None | None | Add-on config files |
| **Network-transparent HW** | Any container accesses any device on any node | No | No | No |
| **Dynamic HW scheduling** | Auto-labels Swarm nodes on hotplug | No | No | No |
| **Offline operation** | Full (local registry + AP) | No | No | Partial |
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

**Architecture difference:** CubeOS-Edge uses **K3s** instead of Docker Swarm for orchestration. While Swarm is ideal for single-node Raspberry Pi deployments (lightweight, simple, Docker-native), DGX Spark clusters require Kubernetes-level features: Layer 7 routing, GPU scheduling, distributed inference coordination, and multi-node storage. The CubeOS API backend, Vue.js dashboard, and HAL architecture remain the same — the HAL simply gains GPU-specific discovery (nvidia-smi, VRAM, CUDA capabilities) while the orchestration layer changes underneath.

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
| **HAL** | Go privileged daemon, 80+ REST endpoints, Swagger UI, dynamic Swarm labels, udev hotplug |
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
- [ ] HAL dynamic Swarm node labels (auto-discovery → label management via Docker API)
- [ ] HAL udev hotplug monitoring (device add/remove → label update → service reschedule)
- [ ] HAL network-transparent device access (cross-node REST API for low-bandwidth devices)
- [ ] Dashboard hardware attachment UI (per-app device selection with one-click redeploy)
- [ ] Multi-node HAL aggregation (unified hardware inventory across cluster)
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
  <strong>Built for expeditions, emergencies, and everyone who wants hardware that just works — across every node.</strong>
  <br><br>
  <sub>CubeOS · Apache 2.0 · Forever free · Forever open</sub>
</p>

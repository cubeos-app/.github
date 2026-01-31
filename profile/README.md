<p align="center">
  <img src="https://avatars.githubusercontent.com/u/257679218" alt="CubeOS Logo" width="120">
</p>

<h1 align="center">CubeOS</h1>

<p align="center">
  <strong>What Synology DSM is for NAS, CubeOS is for Raspberry Pi</strong>
</p>

<p align="center">
  <a href="https://cubeos.app">Website</a> •
  <a href="https://github.com/cubeos-app/docs">Documentation</a> •
  <a href="https://github.com/cubeos-app/releases">Download</a>
</p>

---

## About

**CubeOS** is an open-source ARM64 server operating system for Raspberry Pi and single-board computers. It provides a web-based management interface for Docker services, network configuration, and system monitoring — with an **offline-first** design.

### Key Features

- **Web Dashboard** — Manage Docker containers, view logs, monitor system resources
- **WiFi Access Point** — Turn your Pi into a portable hotspot with three network modes
- **App Store** — Install self-hosted apps with one click (CasaOS-compatible)
- **Offline-First** — Local Docker registry with 20+ pre-cached apps
- **VPN Support** — WireGuard, OpenVPN, and per-app Tor routing
- **Storage** — SMB/NFS client for remote shares and backups
- **Hardware Detection** — UPS HAT, NVMe, RTC, I2C sensors
- **White-Label Ready** — Apache 2.0 license, fully customizable

### Target Users

- Self-hosting enthusiasts
- Field deployments and expeditions
- Emergency/disaster response
- Edge computing and IoT
- Anyone who wants Synology-like features on a Pi

---

## Repositories

| Repo | Description |
|------|-------------|
| [**api**](https://github.com/cubeos-app/api) | Go backend — REST API, Docker Swarm orchestration |
| [**dashboard**](https://github.com/cubeos-app/dashboard) | Vue.js 3 frontend — Web management interface |
| [**coreapps**](https://github.com/cubeos-app/coreapps) | Docker Compose configs for system services |
| [**scripts**](https://github.com/cubeos-app/scripts) | Deployment and utility scripts |
| [**releases**](https://github.com/cubeos-app/releases) | Packer configs and SD card images |
| [**docs**](https://github.com/cubeos-app/docs) | User guides and architecture documentation |

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Backend | Go 1.24+, chi router, SQLite |
| Frontend | Vue.js 3, Pinia, Tailwind CSS |
| Orchestration | Docker Swarm (single-node) |
| Networking | Pi-hole, Nginx Proxy Manager, hostapd |
| Target Hardware | Raspberry Pi 5/4, ARM64 SBCs (2GB+ RAM) |

---

## Quick Start

1. **Download** the latest image from [Releases](https://github.com/cubeos-app/releases)
2. **Flash** to SD card using [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
3. **Boot** your Pi and connect to the `CubeOS-Setup` WiFi network
4. **Open** http://cubeos.cube in your browser
5. **Complete** the setup wizard

---

## Contributing

We welcome contributions! Please see our [Contributing Guide](https://github.com/cubeos-app/docs/blob/main/CONTRIBUTING.md) for details.

- Report bugs via [GitHub Issues](https://github.com/cubeos-app/api/issues)
- Join discussions in [GitHub Discussions](https://github.com/orgs/cubeos-app/discussions)

---

## License

CubeOS is open source under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0).

---

<p align="center">
  <sub>Built with ❤️ for the self-hosting community</sub>
</p>

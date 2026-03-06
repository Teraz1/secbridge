# 🔌 SecBridge — Community Security Integration Kits

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CI](https://img.shields.io/github/actions/workflow/status/YOUR_USERNAME/secbridge/validate-kit.yml?label=kit%20validation)](../../actions)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Integrations](https://img.shields.io/badge/integrations-1-blue.svg)](#-available-integrations)

> **"If the vendor won't build the integration, the community will."**

SecBridge is an open-source collection of **drop-in integration kits** for security products that don't natively talk to each other. Each kit is a self-contained, production-ready package that any engineer can deploy on a single Linux VM in minutes — no paid middleware, no consultants, no waiting for a vendor roadmap.

---

## 🎯 The Problem We Solve

Many organizations run mixed-vendor security stacks — an EDR from one vendor, a firewall from another, a SIEM from a third. Vendors build native integrations only for their biggest partners. Everyone else is left with:

- Incomplete documentation
- Custom scripts that break on updates
- Expensive professional services
- Forum threads that go unanswered

SecBridge fixes this — one integration at a time, built and verified by the community.

---

## 📦 Available Integrations

| # | Source Product | Destination | Protocol | Status |
|---|---------------|-------------|----------|--------|
| 001 | [Sangfor NGAF (Firewall)](integrations/sangfor-ngaf-to-sentinelone/) | SentinelOne Singularity SDL | Syslog UDP/TCP | ✅ Stable |
| — | *Your integration here* | *...* | *...* | [Contribute!](CONTRIBUTING.md) |

---

## 🚀 Quick Start

Every kit follows the same 3-step pattern:

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/secbridge.git
cd secbridge/integrations/sangfor-ngaf-to-sentinelone

# 2. Run the installer (prompts for your API key)
sudo bash scripts/install.sh

# 3. Deploy the log parser as a background service
sudo bash scripts/deploy-parser.sh
```

---

## 🖥️ Web UI (Optional)

SecBridge includes an optional web dashboard to manage sources, view the pipeline status, and configure destinations — all without touching the terminal.

```bash
cd secbridge/web
sudo bash install.sh
```

Opens at `http://YOUR_VM_IP:3000`
Default login: **admin / admin**

### What the Web UI includes

| Page | What it does |
|------|-------------|
| **Dashboard** | Live log throughput, source health, recent events |
| **Pipeline Map** | Visual end-to-end flow — see exactly where issues are |
| **Sources** | Add / remove / enable / disable syslog sources |
| **Parsers** | Manage vendor parser files |
| **Destinations** | Configure SentinelOne SDL API key and URL |
| **Settings** | Restart agent, view config file paths |

### Pipeline Map

The Pipeline Map shows your full log flow in one view:

```
Sources → SecBridge → Parsers → Log Files → SentinelOne SDL
```

Each source gets its own row across all stages. Color coded:
- 🟢 Green = healthy and receiving logs
- 🟡 Yellow = active but has parse errors
- ⭕ Grey = inactive / no logs received

### Web UI folder structure

```
web/
├── install.sh          ← one command — installs everything
├── backend.py          ← FastAPI REST API (port 8000)
├── requirements.txt    ← Python deps
├── README.md           ← web-specific docs
└── frontend/
    ├── index.html
    ├── package.json
    ├── vite.config.js
    └── src/
        ├── App.jsx     ← full React UI
        └── main.jsx
```

### Web UI API endpoints

| Method | Endpoint | What it does |
|--------|----------|-------------|
| POST | `/api/login` | Authenticate |
| GET | `/api/sources` | List all sources |
| POST | `/api/sources` | Add new source |
| DELETE | `/api/sources/{id}` | Remove source |
| PATCH | `/api/sources/{id}/toggle` | Enable / disable |
| POST | `/api/apply` | Apply config changes |
| GET | `/api/status` | Agent status + log sizes |
| GET | `/api/logs/{product}` | Tail a log file |
| GET | `/api/destination` | Read SDL credentials |
| POST | `/api/destination` | Save SDL credentials |
| POST | `/api/restart` | Restart Scalyr Agent |

> See [web/README.md](web/README.md) for full web UI documentation.

---

## ⚙️ Managing Multiple Sources (CLI)

SecBridge supports multiple firewall/security devices forwarding to the **same Linux collector VM** — each on its own port, with its own log file and firewall rules.

All sources are managed from a single config file: `config/sources.json`

### Add a New Source

```bash
# Interactive wizard
bash scripts/manage-sources.sh add

# Apply changes → opens ports + regenerates agent.json
sudo bash scripts/manage-sources.sh apply
```

### List All Sources

```bash
bash scripts/manage-sources.sh list
```

```
── Configured Sources ──

  ID   STATUS   NAME                PORT   PROTO  ALLOWED IPS
  ---  -------  ------------------  -----  -----  -----------
  001  enabled  Sangfor NGAF        514    udp    any
  002  enabled  Fortinet FortiGate  5140   udp    192.168.10.1
  003  enabled  Cisco ASA           5141   tcp    10.0.0.1
```

### Other Commands

```bash
bash scripts/manage-sources.sh status          # live port check + log tail
sudo bash scripts/manage-sources.sh remove 002 # disable a source
```

### How Multiple Sources Work

```
Sangfor NGAF ──── UDP:514  ──┐
Fortinet FGT ──── UDP:5140 ──┤  Linux Collector VM
Cisco ASA    ──── TCP:5141 ──┘  (manage-sources.sh apply)
                                 ├── agent.json (auto-regenerated)
                                 ├── firewall rules (auto-applied)
                                 └── per-source log files
                                        │ HTTPS
                                        ▼
                              SentinelOne SDL
                              (parse fields in SDL console)
```

Each source gets:
- Its own syslog port (must be unique)
- Its own log file `/var/log/scalyr-agent-2/<product>.log`
- Optional IP allowlist (restrict syslog to specific device IPs)

---

## 📁 Kit Structure

Every integration kit follows this layout:

```
integrations/<source>-to-<destination>/
├── scripts/
│   ├── install.sh          ← one-command installer (Ubuntu + Rocky Linux)
│   ├── deploy-parser.sh    ← deploys parser as systemd service
│   └── test-syslog.sh      ← sends test logs to verify pipeline
├── config/
│   └── <product>-parser.json
├── parser/
│   └── <product>_parser.py ← log format → JSON parser (optional)
└── docs/
    ├── README.md            ← full setup guide
    └── SAMPLE_LOGS.md       ← real log examples
```

---

## 🗺️ Integration Roadmap — Community Wanted

### 🔥 High Priority
- [ ] Palo Alto PAN-OS → SentinelOne SDL
- [ ] Fortinet FortiGate → SentinelOne SDL
- [ ] Cisco ASA / FTD → SentinelOne SDL
- [ ] Sangfor NGAF → Microsoft Sentinel
- [ ] Sangfor NGAF → Elastic SIEM

### 🔶 Medium Priority
- [ ] Sophos XG Firewall → SentinelOne SDL
- [ ] WatchGuard Firebox → SentinelOne SDL
- [ ] Huawei USG Firewall → SentinelOne SDL
- [ ] Check Point → SentinelOne SDL
- [ ] Sangfor Endpoint Secure → SentinelOne SDL

### 🔵 Any Firewall → Other SIEM Targets
- [ ] → Wazuh
- [ ] → Graylog
- [ ] → QRadar
- [ ] → Splunk

> 💡 Don't see your product? [Open an integration request](../../issues/new?template=integration-request.md)

---

## 🤝 Contributing

We welcome contributions from anyone who has solved an integration problem and wants to share it.

Read the full guide: [CONTRIBUTING.md](CONTRIBUTING.md)

**Quick checklist before your PR:**
- [ ] `bash -n scripts/install.sh` passes
- [ ] `python3 -m py_compile parser/<product>_parser.py` passes
- [ ] `python3 parser/<product>_parser.py --test` passes with real log samples
- [ ] Tested on Ubuntu 22.04/24.04 or Rocky Linux 9
- [ ] No hardcoded IPs or credentials

---

## 💬 Community & Support

- **Questions / Ideas:** [GitHub Discussions](../../discussions)
- **Bug Reports:** [GitHub Issues](../../issues)
- **New Integration Requests:** [Open a request](../../issues/new?template=integration-request.md)

---

## ⚖️ License

MIT License — free to use, modify, and distribute. See [LICENSE](LICENSE).

---

*Built by the community, for the community. Vendors don't have to be gatekeepers.*

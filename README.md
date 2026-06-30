<p align="center">
  <img src="RM_HEADER.png" alt="Local Agent Header" width="900" />
<p align="center">
  <div style="border-radius: 15px; overflow: hidden; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); transition: transform 0.3s ease-in-out; display: inline-block;">
    <img src="RM_HEADER.png" alt="Local Agent Header" width="900" style="display: block; border-radius: 15px;" />
  </div>
</p>
</p>
<p align="center">
  <strong>A Docker-native AI agent for your editor — local model serving, scoped security scanning, and MCP tool access, without your code leaving your machine.</strong><br>
  <em>Version: 1.0 (Public Preview)</em>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License: MIT"/></a>
  <a href="package.json"><img src="https://img.shields.io/badge/node-%E2%89%A520%20LTS-339933.svg" alt="Node"/></a>
  <a href="https://marketplace.visualstudio.com/items?itemName=YOUR_PUBLISHER_ID.local-agent"><img src="https://img.shields.io/visual-studio-marketplace/v/YOUR_PUBLISHER_ID.local-agent.svg" alt="VS Code Marketplace"/></a>
  <a href="https://github.com/YOUR_ORG/local-agent/actions"><img src="https://github.com/YOUR_ORG/local-agent/actions/workflows/ci.yml/badge.svg" alt="CI"/></a>
  <a href="https://github.com/YOUR_ORG/local-agent/releases"><img src="https://img.shields.io/github/v/release/YOUR_ORG/local-agent" alt="Release"/></a>
  <a href="#roadmap--partner-access-nda"><img src="https://img.shields.io/badge/status-active--development-orange.svg" alt="Status"/></a>
</p>

<p align="center">
  <a href="#overview">Overview</a> •
  <a href="#core-capabilities">Core Capabilities</a> •
  <a href="#architecture">Architecture</a> •
  <a href="#security--privacy-by-design">Security &amp; Privacy</a> •
  <a href="#roadmap--partner-access-nda">Roadmap &amp; Partner Access</a> •
  <a href="#installation">Installation</a>
</p>

---

> *"AI agents are only as trustworthy as the boundary around them. Local Agent's bet is that the boundary should be your machine, your Docker daemon, and your own running containers — not a vendor's cloud."*
>
> — The Local Agent Team

---

> **This README documents Local Agent's public feature set as of this writing. Capabilities that are still under active development or governed by partner agreements are labeled accordingly below rather than described in technical detail — see [Roadmap & Partner Access](#roadmap--partner-access-nda).**

Local Agent is a VS Code sidebar extension that gives a locally running LLM real, scoped access to your development environment: it can stand up your project inside Docker, call tools over the Model Context Protocol (MCP), write and verify its own work, and check the container it just built for common security misconfigurations — all without your code leaving your machine unless you explicitly choose to deploy it. It is built from two complementary systems:

1. **The Agent Runtime** — Docker-managed container lifecycle, local model serving via Ollama, MCP tool calling, streaming chat, and a goal-driven autonomous loop that can write code, run it, and iterate until it passes.
2. **The Safeguard & Audit Layer** — a security scanner that is hard-scoped to the container you just launched, PIN-secured encrypted project mirroring, and a running session log for after-the-fact review.

**Current status:** the Agent Runtime and the Safeguard & Audit Layer are built and working end-to-end. Local, single-machine deployment is live today. Multi-cloud deployment automation, our tensor-level model-compression research, and browser-based companion experiences are at different stages of development — see the status legend below for exactly which is which.

```text
agent runtime          -> Docker lifecycle (Mirror & Manage), Ollama serving, MCP tool calling, chat : SHIPPED
autonomous loop         -> goal-driven write / run / verify / iterate cycle                            : SHIPPED
safeguard scanning      -> OWASP ZAP baseline scan, hard-scoped to your own running container only      : SHIPPED
encrypted mirroring     -> PIN-secured local project mirroring; per-session audit log                   : SHIPPED
local deployment        -> single-machine serving                                                       : SHIPPED
spectral compression    -> tensor-level model compression research ("Spectrally")                       : NDA-TRACK
extended deployment     -> multi-cloud targets & CI/CD pipelines                                         : NDA-TRACK
browser integration     -> directional, shaping with early design partners                               : EXPLORATORY
```

| | What it does |
|---|---|
| **Mirror a Project** | Point Local Agent at a local folder; it generates a Dockerfile, builds the image, and runs your project in a container with the agent attached. |
| **Manage Containers** | Attach the agent to containers you're already running, via MCP, without mirroring a new project. |
| **Chat & Tools** | Streaming chat backed by your chosen local Ollama model, with MCP tools you explicitly enable per session — nothing is granted by default. |
| **Autonomous Loop** | Hand the agent an objective — e.g. "write tests for this file and run them until they pass" — and watch it iterate in the same chat thread. |
| **Safeguard** | A self-scoped OWASP ZAP baseline scan against the container you just launched. There is no field to point it anywhere else. |
| **Report** | A running session/audit log of what the agent did, for after-the-fact review. |

---

## Overview

Most AI coding agents today are cloud-hosted: your code and the agent's actions live on someone else's infrastructure. Local Agent inverts that — the model runs on your machine via Ollama, the agent's actions happen inside a Docker container you control, and the only thing that ever leaves your machine is what you explicitly deploy. The extension itself is a thin, sandboxed WebView sidebar talking to a Node.js/TypeScript extension host, which talks to the Docker Engine API directly (no shelling out to the Docker CLI).

## Core Capabilities

- **Two workflows.** *Mirror a project* auto-generates a Dockerfile, builds it, and runs your codebase in a container with the agent attached. *Manage containers* instead connects the agent to infrastructure you're already running.
- **Local model serving.** Local Agent auto-discovers running Ollama containers and their available models — no manual endpoint configuration.
- **MCP tool access**, enabled per tool, per session, from the sidebar — you choose what the agent can touch.
- **An autonomous write-and-verify loop.** The agent can be handed an objective (read a file, write tests, run the suite, fix failures, repeat) and will work it end-to-end inside the same chat thread.
- **Safeguard**, a baseline OWASP ZAP security scan that can only ever target the container Local Agent just built for you.
- **PIN-secured project mirroring**, encrypting access to a mirrored folder behind a 6-digit PIN you set before the container is built.
- **A session audit log**, recording what the agent did for later review.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    VS Code IDE                       │
│                                                       │
│  ┌──────────────┐      ┌─────────────────────────┐  │
│  │   Activity   │      │   Extension Host         │  │
│  │   Bar Icon   │─────▶│   (Node.js / TypeScript) │  │
│  └──────────────┘      │                           │  │
│                        │  extension.ts             │  │
│  ┌──────────────┐      │  DockerManager.ts         │  │
│  │   Sidebar    │◀────▶│  SidebarProvider.ts       │  │
│  │  (WebView)   │      └──────────┬────────────────┘  │
│  │  main.js     │                 │                   │
│  │  styles.css  │                 │ dockerode         │
│  └──────────────┘                 ▼                   │
└────────────────────────────────────┼──────────────────┘
                                     │
                         ┌───────────▼───────────┐
                         │   Docker Engine API    │
                         │  /var/run/docker.sock  │
                         └───────────┬────────────┘
                                     │
                         ┌───────────▼───────────┐
                         │   Docker Containers    │
                         │ (Ollama, mirrored app) │
                         └────────────────────────┘
```

## Security & Privacy by Design

Two choices here are deliberate. Safeguard's OWASP ZAP scan has no field for an arbitrary target URL — it can only ever point at the container Local Agent just built, so it's a self-check, not a general-purpose scanning tool. And Mirror workflows can be protected behind a 6-digit PIN, set before the container is built, used to encrypt access to your mirrored project folder. Local model serving never requires sending code anywhere; anything beyond your own machine is opt-in. Partners evaluating Local Agent for environments with formal security or compliance requirements can request deeper architecture documentation under NDA — see below.

## Roadmap & Partner Access (NDA)

Local Agent is under active development. The items below are real workstreams, not placeholders — but implementation details and timelines are kept out of the public README for competitive and partnership reasons. Maturity is labeled using the legend below; reach out to discuss any 🟡 or ⚪ item under NDA.

- 🟢 **Shipped** — built, working, and described above.
- 🟡 **NDA-Track** — active development; we'll walk partners and design customers through specifics under NDA.
- ⚪ **Exploratory** — directional, not yet built; we're shaping it together with early design partners.

| Capability | Status |
|---|---|
| Spectral compression — tensor-level model compression for faster local inference | 🟡 NDA-Track |
| Extended deployment — multi-cloud targets and CI/CD pipelines (local single-machine deployment is shipped today) | 🟡 NDA-Track |
| Browser integration | ⚪ Exploratory |

Interested in early access, a technical briefing, or helping shape the roadmap? Reach out at **[contact email]**.

## Installation

Search "Local Agent" in the VS Code Marketplace, or install a `.vsix` from the [Releases](https://github.com/YOUR_ORG/local-agent/releases) page. Requires Docker Desktop (or another Docker Engine–compatible runtime) running locally, and at least one model pulled into a reachable Ollama instance.

```bash
code --install-extension local-agent-0.0.1.vsix
```

## Contributing

Local Agent is in active development. Bug reports and feature discussion are welcome via GitHub Issues. See `CONTRIBUTING.md` for the development workflow (TypeScript + esbuild; press F5 to launch the Extension Development Host).

## License

MIT
# Abejar Post-Quantum Cryptography VPN Router

<p align="center">
  <strong>Cloudflare Tunnel PQC Edition</strong>
</p>

<p align="center">
  <a href="https://github.com/vinzabe/abejar-pqc-vpn-cloudflare-tunnel/actions/workflows/ci.yml"><img src="https://github.com/vinzabe/abejar-pqc-vpn-cloudflare-tunnel/actions/workflows/ci.yml/badge.svg" alt="CI"></a>
  <a href="https://github.com/vinzabe/abejar-pqc-vpn-cloudflare-tunnel/pkgs/container/abejar-pqc-cloudflared"><img src="https://img.shields.io/badge/Docker-ghcr.io-blue?logo=docker" alt="Docker"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Commercial-red" alt="License"></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/PQC-Cloudflare%20TLS%201.3-purple?style=for-the-badge" alt="PQC Cloudflare"/>
  <img src="https://img.shields.io/badge/Security-Zero--Trust-green?style=for-the-badge" alt="Zero Trust"/>
</p>

---

## Overview

**Abejar PQC VPN Router (Cloudflare Tunnel Edition)** focuses post-quantum security on the Cloudflare Tunnel ingress layer, using TLS 1.3 with Kyber key agreement.

This variant is ideal when your primary concern is **securing public-facing services** with quantum-resistant encryption at the edge.

---

## Post-Quantum Cryptography Security

### Why PQC at the Edge?

The edge is where your services first meet the public internet. Securing this point with post-quantum cryptography ensures:

- **Client-to-Edge Protection**: User connections are quantum-resistant from day one
- **Future-Proof TLS**: Protected against "harvest now, decrypt later" attacks
- **Zero Trust**: No exposed ports on your infrastructure

### Cloudflare's PQC Implementation

Cloudflare has deployed post-quantum TLS across their entire network using:

| Component | Implementation |
|-----------|----------------|
| Key Agreement | X25519Kyber768Draft00 (Hybrid) |
| Cipher Suite | TLS_AES_256_GCM_SHA384 |
| Protocol | TLS 1.3 |
| Security Level | NIST Level 3 (Kyber-768) |

### Security Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    PQC TLS Connection Flow                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   User Browser                    Cloudflare Edge               │
│   ┌──────────────┐               ┌──────────────┐              │
│   │  TLS 1.3     │───────────────│  PQC TLS     │              │
│   │  Client      │  Kyber-768    │  Termination │              │
│   │  (PQC)       │  Key Exchange │              │              │
│   └──────────────┘               └──────┬───────┘              │
│                                         │                       │
│                              Tunnel Protocol                    │
│                                         │                       │
│                                         ▼                       │
│                                  ┌──────────────┐              │
│                                  │  Cloudflared │              │
│                                  │  Container   │              │
│                                  └──────┬───────┘              │
│                                         │                       │
│                                    localhost                    │
│                                         │                       │
│                                         ▼                       │
│                                  ┌──────────────┐              │
│                                  │ Application  │              │
│                                  └──────────────┘              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Features

### Security
- **PQC TLS at Edge** - Quantum-resistant encryption from user to Cloudflare
- **Cloudflare Tunnel** - Secure ingress without exposed ports
- **Zero Trust** - No public IP exposure on your infrastructure
- **Automatic Certificates** - Managed TLS certificates with PQC

### Simplicity
- **Easy Setup** - Works with existing Cloudflare infrastructure
- **No Port Forwarding** - Outbound-only connections
- **Automatic Updates** - Cloudflare handles TLS/PQC updates

### Integration
- **Docker Compose** - One-command deployment
- **VPN Routing** - Optional outbound traffic through VPN
- **Multi-App Support** - Route multiple services through single tunnel

---

## Quick Start

### Prerequisites

- Docker 24.0+
- Docker Compose 2.20+
- Cloudflare account with Tunnel configured
- Cloudflare Tunnel token

### Installation

```bash
# Clone the repository
git clone https://github.com/vinzabe/abejar-pqc-vpn-cloudflare-tunnel.git
cd abejar-pqc-vpn-cloudflare-tunnel

# Run setup script
./scripts/setup.sh

# Configure environment with your Cloudflare Tunnel token
nano .env

# (Optional) Configure WireGuard for outbound VPN
nano vpn-router/config/wireguard/wg0.conf

# Start the tunnel
docker compose up -d

# Verify status
./scripts/vpn-status.sh
```

### Using Pre-built Images

```bash
# Pull the pre-built image
docker pull ghcr.io/vinzabe/abejar-pqc-cloudflared:latest
```

### Verify PQC TLS

You can verify post-quantum TLS is active using:

```bash
# Check with curl (requires curl 8.5+ with PQC support)
curl -v https://your-domain.com 2>&1 | grep -i kyber

# Or use Cloudflare's dashboard to verify tunnel status
```

---

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CF_TUNNEL_TOKEN` | - | **Required**: Cloudflare Tunnel token |
| `CF_TUNNEL_PQC` | `true` | Enable post-quantum TLS |
| `CF_LOG_LEVEL` | `info` | Cloudflared log level |
| `PQC_ENABLED` | `true` | Enable PQC features |
| `PQC_ALGORITHM` | `kyber1024` | PQC algorithm for VPN (if used) |
| `VPN_INTERFACE` | `wg0` | WireGuard interface |
| `VPN_NETWORK_CIDR` | `172.25.0.0/16` | Docker network CIDR |
| `VPN_ROUTER_IP` | `172.25.0.2` | VPN router IP |
| `APP_PORT_START` | `10091` | Starting port for app mappings |
| `APP_PORT_END` | `10100` | Ending port for app mappings |
| `LOG_LEVEL` | `info` | General log level |

### Cloudflare Tunnel Setup

1. Create a tunnel in Cloudflare Zero Trust dashboard
2. Copy the tunnel token
3. Add to your `.env` file:

```bash
CF_TUNNEL_TOKEN=eyJhIjoiYWJjMTIzLi4uIiwidCI6IjEyMzQ1Njc4LTkwYWItY2RlZi4uLiIsInMiOiJhYmNkZWYxMi4uLiJ9
```

4. Configure public hostnames in Cloudflare dashboard

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                       Public Internet                             │
│  ┌────────────────┐                                              │
│  │   Users        │                                              │
│  └───────┬────────┘                                              │
│          │ HTTPS (PQC TLS 1.3 + Kyber-768)                       │
│          ▼                                                        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                 Cloudflare Edge                           │   │
│  │  ┌────────────────┐    ┌─────────────────────────────┐   │   │
│  │  │  CDN / WAF     │    │  PQC TLS Termination        │   │   │
│  │  │  DDoS Protect  │────│  X25519Kyber768Draft00      │   │   │
│  │  └────────────────┘    └─────────────┬───────────────┘   │   │
│  └──────────────────────────────────────┼───────────────────┘   │
│                                          │                        │
│                          Tunnel Protocol │                        │
│                                          │                        │
└──────────────────────────────────────────┼────────────────────────┘
                                           │
┌──────────────────────────────────────────┼────────────────────────┐
│                                          ▼                        │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │                      Docker Host                            │  │
│  │  ┌──────────────────────────────────────────────────────┐  │  │
│  │  │              VPN Router Container                     │  │  │
│  │  │  ┌────────────┐  ┌────────────┐  ┌────────────────┐  │  │  │
│  │  │  │ WireGuard  │  │  Kyber     │  │ Traffic Router │  │  │  │
│  │  │  │ (Optional) │──│  (Opt.)    │──│                │  │  │  │
│  │  │  └────────────┘  └────────────┘  └────────────────┘  │  │  │
│  │  └──────────────────────────┬───────────────────────────┘  │  │
│  │                             │                               │  │
│  │  ┌──────────────────────────┼───────────────────────────┐  │  │
│  │  │            Cloudflared Container                      │  │  │
│  │  │  ┌────────────────┐      │     ┌──────────────────┐  │  │  │
│  │  │  │ Tunnel Client  │──────┴─────│ --post-quantum   │  │  │  │
│  │  │  └────────────────┘            └──────────────────┘  │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  │                             │                               │  │
│  │  ┌──────────────────────────┼───────────────────────────┐  │  │
│  │  │             Application Containers                    │  │  │
│  │  │  ┌────────────┐  ┌────────────┐  ┌────────────────┐  │  │  │
│  │  │  │  Web App   │  │    API     │  │   Database     │  │  │  │
│  │  │  │ :10091     │  │  :10092    │  │  (internal)    │  │  │  │
│  │  │  └────────────┘  └────────────┘  └────────────────┘  │  │  │
│  │  └──────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
```

### Security Layers

| Layer | Protection | Technology |
|-------|------------|------------|
| 1. Edge | DDoS, WAF | Cloudflare |
| 2. TLS | Quantum-resistant encryption | TLS 1.3 + Kyber-768 |
| 3. Tunnel | Secure ingress | Cloudflare Tunnel |
| 4. Network | No exposed ports | Zero Trust |
| 5. Container | Process isolation | Docker |
| 6. VPN (opt.) | Outbound anonymity | WireGuard + Kyber |

---

## PQC TLS Details

Cloudflare's post-quantum TLS implementation uses:

| Parameter | Value |
|-----------|-------|
| Key Agreement | X25519Kyber768Draft00 |
| Cipher Suite | TLS_AES_256_GCM_SHA384 |
| Protocol | TLS 1.3 |
| PQC Algorithm | Kyber-768 (NIST Level 3) |
| Classic Algorithm | X25519 (Curve25519) |
| Mode | Hybrid (both must be broken) |

### Browser Support

Modern browsers automatically negotiate PQC TLS when available:

| Browser | PQC Support |
|---------|-------------|
| Chrome 116+ | Yes (Kyber-768) |
| Firefox 123+ | Yes (Kyber-768) |
| Edge 116+ | Yes (Kyber-768) |
| Safari | Not yet |

---

## Documentation

| Document | Description |
|----------|-------------|
| [SECURITY.md](SECURITY.md) | Security analysis and threat model |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | System architecture details |
| [docs/CLOUDFLARE_SETUP.md](docs/CLOUDFLARE_SETUP.md) | Cloudflare Tunnel configuration |
| [docs/INSTALLATION.md](docs/INSTALLATION.md) | Detailed installation guide |

---

## License

This software is commercially licensed. Pre-built Docker images are provided for evaluation.

For **full source code**, **enterprise licensing**, and **support**:

| | |
|---|---|
| **Email** | grant@abejar.net |
| **Subject** | Abejar PQC VPN - License Inquiry |

---

<p align="center">
  <sub>Copyright 2024 Abejar. All rights reserved.</sub>
</p>

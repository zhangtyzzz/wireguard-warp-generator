# WARP WireGuard Config Generator

Generate WireGuard configurations for Cloudflare WARP.

## Web App

**[https://lanrat.github.io/wireguard-warp-generator/](https://lanrat.github.io/wireguard-warp-generator/)**

A browser-based tool that generates configs entirely client-side. Pick a tunnel protocol and an output format:

| Protocol | Output | Use |
|----------|--------|-----|
| WireGuard | `warp.conf` | Official WireGuard clients, with a QR code for mobile import |
| WireGuard | `warp.yaml` | Standalone Clash / Stash profile, carrying the `reserved` bytes |
| WireGuard | `warp.stoverride` | Adds the proxy to the profile Stash already has loaded |
| MASQUE | `warp.yaml` / `warp.stoverride` | `type: masque` proxy on UDP 443, needs Stash 3.6+ / macOS 4.3+ |

MASQUE is the protocol the official WARP client uses: CONNECT-IP over HTTP/3 on port 443, served
from Cloudflare's `162.159.198.0/24` range rather than WireGuard's UDP 2408. An optional WARP+
license key attaches your subscription to the generated device, and an optional device name is
stored on it.

## Shell Script

Command-line tool for generating WARP configs. See [scripts/README.md](scripts/README.md) for usage.

```bash
# Basic usage
./scripts/warp-register.sh > warp.conf

# With QR code and account info
./scripts/warp-register.sh --qr --info > warp.conf

# With a WARP+ license key
./scripts/warp-register.sh --key xxxxxxxx-xxxxxxxx-xxxxxxxx > warp.conf

# With a device name
./scripts/warp-register.sh --name my-laptop > warp.conf
```

## How It Works

1. Generates a WireGuard keypair locally
2. Registers the public key with Cloudflare's WARP API
3. Names the device and attaches a WARP+ license key, if either was provided
4. Outputs a complete WireGuard configuration

## Reserved Bytes

Cloudflare puts a 24-bit `client_id` in WireGuard's three reserved header bytes and load balances
packets by hashing it, so a tunnel that sends zeros there can lose its server affinity and stall.
Both tools report the value (`ClientID` comment and account info) and the web app writes it into the
`reserved` field of the Stash / Clash profile. The official WireGuard clients cannot send these
bytes; userspace clients such as Stash, mihomo, sing-box and Xray can.

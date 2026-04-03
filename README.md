# homelab-jumpbox

A lightweight, secure SSH jump box for homelab remote access — no VPN platform required.

## What This Is

A containerized OpenSSH server that acts as a hardened entry point into your homelab. You expose only this container to the internet. From it, you can SSH into any internal server on your LAN. As a bonus, you can tunnel a SOCKS5 proxy through the same connection to reach internal web UIs from your browser.

```
Internet → Router (non-standard port) → jumpbox container → Internal servers
```

No Tailscale. No WireGuard config. No extra platform. Just SSH.

---

## Features

- Key-only authentication (no passwords)
- Non-standard external port to reduce scanner noise
- TCP forwarding scoped to your user only
- SOCKS5 browser proxy through the same SSH connection
- ProxyJump config for transparent one-command hops to internal servers
- Works with Docker or rootful Podman

---

## Requirements

- A server on your LAN running Docker or rootful Podman
- A domain name pointed at your home IP (optional but recommended)
- Port forwarding access on your router
- SSH keys on your client machines

---

## Quick Start

### 1. Clone the repo

```bash
git clone https://github.com/distantgeek/homelab-jumpbox
cd homelab-jumpbox
```

### 2. Deploy the container

```bash
# Create config directory
sudo mkdir -p /opt/jumpbox/config

# Deploy
docker compose up -d
# or
podman compose up -d
```

### 3. Add your public keys

```bash
echo "ssh-ed25519 AAAA...yourkey YourLabel" | sudo tee -a /opt/jumpbox/config/authorized_keys
```

Restart the container to pick up the keys:

```bash
docker restart jumpbox
# or
podman restart jumpbox
```

### 4. Open your firewall (Fedora/firewalld)

```bash
sudo firewall-cmd --add-port=2222/tcp --permanent
sudo firewall-cmd --reload
```

### 5. Forward your router

Forward a high non-standard port on your router to your server's LAN IP on port 2222.

Example: `51022 (external) → 192.168.x.x:2222`

### 6. Configure your SSH client

Copy `config/ssh_config.example` to `~/.ssh/config` on each client machine and fill in your IPs.

### 7. Set up your tunnel alias

Add to `~/.bashrc` or `~/.zshrc`:

```bash
alias homelab-proxy="ssh -N -q homelab-jump"
```

---

## Usage

### SSH into the jumpbox

```bash
ssh -p 2222 yourusername@yourdomain-or-ip
```

### Hop directly to an internal server

```bash
ssh fedora-server   # uses ProxyJump automatically via ~/.ssh/config
```

### Start the SOCKS5 browser tunnel

```bash
homelab-proxy   # runs in foreground, Ctrl+C to stop
```

Then configure your browser to use `127.0.0.1:9050` as a SOCKS5 proxy. See [docs/foxyproxy.md](docs/foxyproxy.md) for Firefox setup.

---

## Directory Structure

```
homelab-jumpbox/
├── compose.yml                 # Container definition
├── config/
│   ├── ssh_config.example      # Client-side ~/.ssh/config template
│   └── sshd_config.snippet     # Server-side sshd additions
├── docs/
│   ├── foxyproxy.md            # Firefox SOCKS5 proxy setup
│   ├── hardening.md            # Security recommendations
│   └── router-forwarding.md    # Router port forward guidance
└── README.md
```

---

## Security Notes

- The jumpbox has no `sudo` access and no shell privileges beyond SSH
- TCP forwarding is scoped to your user only via `Match User` in `sshd_config`
- Use a non-standard external port — eliminates the majority of automated scanners
- Consider geo-blocking or IP whitelisting on your router for the forwarded port
- See [docs/hardening.md](docs/hardening.md) for the full checklist

---

## License

MIT

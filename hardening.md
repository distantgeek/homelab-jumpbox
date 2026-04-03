# Hardening Guide

## Jump Container

| Setting | Value | Why |
|---|---|---|
| `PASSWORD_ACCESS` | `false` | Eliminates brute-force surface entirely |
| `SUDO_ACCESS` | `false` | Container user cannot escalate privileges |
| `AllowTcpForwarding` | Scoped to user via `Match User` | Limits forwarding to your account only |
| Published port | Non-standard (e.g. 2222) | Kills most automated scanners |

---

## Router

- Forward a **high non-standard external port** to the container's port (e.g. `51022 → 2222`)
- If your router supports it, **geo-block** to your home country
- If your IPs are stable, **whitelist your known external IPs** on the forwarded port

---

## Target Servers (Fedora, TrueNAS, etc.)

Restrict SSH access so only connections from your LAN (where the jumpbox host lives)
are accepted for your management user. In `/etc/ssh/sshd_config` on each target:

```
AllowUsers yourusername@192.168.x.x
```

This means even if someone got into the jumpbox container, they could only reach
target servers from your LAN — not from an arbitrary internet IP.

Enable key-based auth only on target servers:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

---

## SSH Keys

- Use **ed25519** keys — smaller and faster than RSA
- Use a **passphrase** on your client keys
- Keep one key per client machine — don't share the same key across devices
- If a device is lost or compromised, remove its key from `authorized_keys` and restart the container

---

## Why Not ForwardAgent?

Agent forwarding passes your local SSH agent socket through the jump connection.
While convenient, it means anyone with root on the jumpbox could use your agent
to authenticate as you to downstream servers while your session is active.

ProxyJump (used in this setup) is strictly safer — your client handles all
authentication directly, and the jumpbox never sees your key material.

---

## Optional: Fail2ban on Target Servers

If your target servers are ever reachable from the jump container, adding fail2ban
provides a backstop against brute-force attempts from inside the tunnel:

```bash
sudo dnf install fail2ban        # Fedora
sudo systemctl enable --now fail2ban
```

Default config monitors `/var/log/auth.log` and bans after 5 failed attempts.

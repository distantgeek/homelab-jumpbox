# Router Port Forwarding

## What to Forward

| External Port | Internal IP | Internal Port |
|---|---|---|
| 51022 (or any high port) | Your jumpbox host LAN IP | 2222 |

Only one port forward is needed. Everything — SSH hops, SOCKS5 tunnel — flows
through this single entry point.

---

## General Steps

The exact UI varies by router firmware, but the concept is the same:

1. Find **Port Forwarding** or **Virtual Server** in your router admin UI
2. Create a new rule:
   - **External/WAN Port:** your chosen high port (e.g. `51022`)
   - **Internal/LAN IP:** the LAN IP of the machine running the jumpbox container
   - **Internal Port:** `2222`
   - **Protocol:** TCP
3. Save and apply

---

## ASUS Routers (AiMesh / Stock / Merlin)

`Advanced Settings → WAN → Virtual Server / Port Forwarding`

- Service Name: `jumpbox`
- External Port: `51022`
- Internal IP: your server LAN IP
- Internal Port: `2222`
- Protocol: TCP

---

## Dynamic DNS

If your ISP assigns a dynamic external IP, set up DDNS so your domain always
points to your current IP. Options:

- **Cloudflare** with `favonia/cloudflare-ddns` container (recommended)
- Your router's built-in DDNS (DynDNS, No-IP, etc.)
- Most ASUS routers have a free `asuscomm.com` DDNS option built in

---

## Testing the Forward

Once the forward is set up, test from outside your LAN (mobile hotspot works):

```bash
ssh -p 51022 yourusername@yourdomain.com
```

Or test with nc first to confirm the port is reachable before trying SSH:

```bash
nc -zv yourdomain.com 51022
```

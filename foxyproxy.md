# Firefox SOCKS5 Proxy Setup with FoxyProxy

FoxyProxy is an open source FireFox extension that lets you toggle the SOCKS5 tunnel on and off from a browser icon
without changing system-wide proxy settings. Normal internet traffic goes direct — only your homelab LAN addresses route through the tunnel.

---

## Installation

Install **FoxyProxy Standard** from the Firefox Add-ons store:
https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/

---

## Configuration

1. Click the FoxyProxy icon in the Firefox toolbar
2. Click **Options**
3. Click **Add** to create a new proxy
4. Fill in:
   - **Title:** Homelab Tunnel
   - **Type:** SOCKS5
   - **Hostname:** `127.0.0.1`
   - **Port:** `9050`
5. Click **Save**

---

## Pattern Rules (Recommended)

Set a pattern so only LAN addresses route through the proxy.
Normal internet traffic continues to go direct.

1. In the proxy entry, click **Add Pattern**
2. Add the following patterns as they are applicable to your network:

| Pattern | Type | Matches |
|---|---|---|
| `192.168.*` | Wildcard | All LAN addresses |
| `10.*` | Wildcard | 10.x.x.x ranges |
| `172.16.*` | Wildcard | 172.16.x.x ranges |

---

## Usage

1. Start the tunnel in a terminal:
   ```bash
   homelab-proxy
   ```
2. Click the FoxyProxy icon and select **Homelab Tunnel**
3. Browse to any internal address, e.g. `http://192.168.x.x:8096`
4. When done, click FoxyProxy icon → **Disable** and `Ctrl+C` the tunnel

---

## Testing

With the tunnel running and FoxyProxy enabled, test with curl:

```bash
curl --socks5-hostname 127.0.0.1:9050 http://192.168.x.x:8096
```

A response (even empty) confirms the tunnel is routing correctly.

# My Setup

**Devices:** 
- Medusa (ROG Strix G16, CachyOS — always-on server + gaming rig) 
- Aqua (MacBook Air M4 — remote dev + Minecraft hosting via OrbStack) 
- Android (mobile ops via Termux)

---

## 1. Software — per device

- **Medusa:** 
  - [VirtualBox](./Guides/VirtualBox.md)
  - Zed
  - Ranger
  - Podman
  - Podman-Desktop
  - Docker
  - Docker-Compose
  - LazyDocker
  - Zathura
  - TailScale
  - LocalSend
  - Obs-Studio
  - Btop
- **Aqua:** 
  - TailScale
  - Zed
  - OrbStack
  - LocalSend
  - Gimp
  - Davinci Resolve
  - Obs-Studio
- **Android:** 
  - TailScale
  - LocalSend
  - F-Droid
  - Termux
    - Tmux
    - OpenSSH

---

## 3. Network layer: Tailscale

### Core config
- **MagicDNS:** on (admin console → DNS) — reach Medusa as `medusa`, no raw 100.x IP needed
- **Key expiry:** disabled for **Medusa only**.

### Tailscale SSH (no per-device key management)
Enable on Medusa without disrupting anything running:
```bash
sudo tailscale set --ssh
```
ACL policy — admin console → Access controls, add an `ssh` block (separate from general network ACLs):
```json
"ssh": [
  {
    "action": "accept",
    "src": ["you@example.com"],
    "dst":    ["autogroup:self"],
	  "users":  ["autogroup:nonroot", "root", "$USER"]
  }
]
```
Swap `you@example.com` for your real Tailscale login. Result: `ssh vincenzo@medusa` works from Aqua or Termux, no keys generated or distributed anywhere.

---

## 4. Per-device access

### Android (Termux) — connected
- On Medusa side: `tmux new -s home` gives a persistent session — lock the phone, lose signal, reattach later with `tmux attach -t home` and everything's still running.
- Same session works for `VBoxManage` too.

### Aqua — SSH config + Zed remote dev + deployment
```
# ~/.ssh/config
Host medusa
    HostName medusa
    User $USER
```
- **Zed remote dev:** command palette → remote-projects action → "Connect New Server" → `medusa` (needs Zed 0.159+). UI stays local on Aqua; language servers/terminal/tasks run on Medusa.

---

## 5. Container engines: the split

| | Docker | Podman |
|---|---|---|
| **Role** | homelab / self-hosted services | personal projects |
| **Why** | nearly every Jellyfin/Navidrome/Immich example online assumes Docker Compose | rootless by default — matches security posture for your own exposed work |

NVIDIA GPU passthrough (Container Toolkit + CDI) works on both now with roughly equal setup effort — not a deciding factor either way anymore.

**Exception:** the Minecraft server (§8) runs on Aqua via OrbStack.

---

## 6. Cockpit — plugins in play

| Plugin | Purpose |
|---|---|
| **cockpit-storaged** | mount/unmount drives, incl. external + LUKS-encrypted |
| **cockpit-file-sharing** | Samba/NFS share management (NAS role) |
| **cockpit-compose** | one view for Docker Compose + Podman Compose stacks together |
| cockpit-podman + cockpit-dockermanager | fallback if cockpit-compose proves too rough |

Cockpit exposed over the tailnet only, never funneled publicly:
```bash
tailscale serve --bg https+insecure://localhost:9090
```

### External HDD via cockpit-storaged
- Plug in → shows under **Drives** on the Storage page
- LUKS passphrase prompt appears before Mount is available (it's the encrypted one)
- Mount / Unmount buttons handle the rest
- **Known quirk:** some USB docks/enclosures aren't auto-detected by Cockpit's device list even though the OS mounts them fine. Fallback, same backend as the GUI:
```bash
udisksctl unlock -b /dev/sdX1   # if LUKS
udisksctl mount -b /dev/sdX1
udisksctl unmount -b /dev/sdX1
```

---

## 7. Homelab services — planned, not yet deployed

- **Video:** Jellyfin (Docker)
- **Music:** Navidrome (Docker)
- **Photos / other:** Immich (Docker)
- **NAS:** native Samba (not containerized — avoids UID/GID permission mapping pain), managed through cockpit-file-sharing

---

## 8. Minecraft server — hosted for friends

**Host:** Aqua, via OrbStack.

### Access model: sharing, not Funnel
Tailscale Funnel only proxies HTTP/HTTPS — Minecraft's raw TCP can't ride on it at all, confirmed directly in Tailscale's own example repo for self-hosting Minecraft. The right tool is Tailscale's **device sharing** feature: it gives a specific person access to exactly one shared device and nothing else in the tailnet — no tags, no groups, no visibility into anything else running. Narrower and safer than public exposure would've been anyway.

### Setup: dedicated Tailscale identity via sidecar
The Minecraft container shares a Tailscale container's network namespace, so it joins the tailnet as its own node (`minecraft`) — completely separate from Aqua's own identity. Friends shared this node see only "minecraft," nothing else.

My minecraft container with my custom modpack is hosted on github and can be found [here](https://github.com/voidrc/Void-s-Adventure).

> Auth key: admin console → Settings → Keys — non-ephemeral, so the node's identity persists across restarts.

---

## 9. VM management on Medusa — controlling from Aqua

> **Hypervisor: VirtualBox.**

### Lifecycle control
Same SSH connection as everything else — `VBoxManage` over `ssh medusa`:
```bash
VBoxManage list vms
VBoxManage startvm kali --type headless
VBoxManage controlvm kali poweroff
```

### Graphical console: VRDE, not VNC/SPICE
VirtualBox's equivalent of a VNC/SPICE console is **VRDE** (VirtualBox Remote Desktop Extension) — RDP-based, works with any standard RDP client. Requires the Extension Pack (separate download from base VirtualBox, free for personal use).

Per VM, on Medusa:
```bash
VBoxManage modifyvm kali --vrde on --vrde-port 3390 --vrdeaddress <medusa-tailscale-ip>
VBoxManage modifyvm parrot-htb --vrde on --vrde-port 3391 --vrdeaddress <medusa-tailscale-ip>
# distinct port per VM — 3389 only serves one at a time
```
Get Medusa's Tailscale IP with `tailscale ip -4`. The bind address wants a literal IP; connecting *from* Aqua can still use `medusa:3390` via MagicDNS.

On Aqua: **Windows App** (Microsoft's Remote Desktop client, Mac App Store) → add PC → `medusa:3390`.

Tighter alternative: bind VRDE to `127.0.0.1` instead and SSH-tunnel it (`ssh -L 3390:localhost:3390 medusa`) rather than binding straight to the Tailscale interface.

---

## 10. Open items

- [ ] 32GB DDR5 RAM upgrade for Medusa
- [ ] Deploy Jellyfin / Navidrome / Immich under Docker
- [ ] Generate Tailscale auth key and share the `minecraft` node with friends
- [ ] Decide on modpack / finalize mod list
- [ ] Install VirtualBox Extension Pack on Medusa
- [ ] Configure VRDE (port + tailscale-bound address) per VM
- [ ] Install Windows App (RDP client) on Aqua
- [ ] Fill in Software placeholder (§2) per device

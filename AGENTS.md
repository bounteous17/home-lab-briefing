# AGENTS.md

Entry point for LLM agents working on the home lab. Start every task here, decide which repository owns the
change, then continue in that repository (read its `README.md`/`AGENTS.md` before acting).

## Repository map

All repos live under `github.com/bounteous17` and are cloned as siblings of this repo
(e.g. `../home-lab-mikrotik-gateway-iac`).

| Repository | Scope | Source of truth for | Live system |
|---|---|---|---|
| `home-lab-briefing` | Hub / welcome page | CIDR plan and per-range address allocation (README "Network plan"), `*.begoodguys.ovh` service catalog, hardware inventory, energy cost, lab-wide security posture, repo map | — |
| `home-lab-mikrotik-gateway-iac` | MikroTik RB2011 gateway (RouterOS 7) | Interfaces & bridges, port map, DHCP servers & reservations, DNS, firewall filter, NAT & hairpin NAT, WireGuard, router management | `ssh bounteous@192.168.11.1` |
| `home-lab-proxmox-ansible` | Proxmox VE cluster (`panda`, `octopus`) | Nodes, quorum, HA, storage, replication, backups, Proxmox networking, every VM/LXC (incl. Nginx Proxy Manager proxy hosts and LXC apps) | `proxmox` MCP server; `ssh root@192.168.11.100/.101` |
| `home-lab-k3s-ansible` | k3s cluster provisioning | Control-plane VM and Raspberry Pi agents: OS prep, k3s install/upgrade, node config | `ssh bounteous@<pi-ip>` |
| `home-lab-k3s-argocd` | Workloads on k3s (GitOps) | Argo CD and the k3s-hosted catalog apps (qBittorrent, Linkwarden, Nextcloud): sync policy, secrets (SOPS + age), app backups | Argo CD / `kubectl` |
| `home-lab-os-ansible` *(planned, not created yet)* | OS patching | Package/OS upgrades for the Pis, the Arch control-plane VM and the Proxmox nodes | — |

### Where does a change go?

| If the task is about… | Work in |
|---|---|
| A new subnet, a new service name, hardware added/removed, power cost | `home-lab-briefing` |
| Port forwards, hairpin NAT, firewall rules, DHCP reservation, VPN peer | `home-lab-mikrotik-gateway-iac` |
| A new LXC/VM, NPM proxy host, backups, HA, Proxmox storage | `home-lab-proxmox-ansible` |
| Adding/replacing a Pi, k3s version upgrades, control-plane VM config | `home-lab-k3s-ansible` |
| Deploying or changing an app on k3s | `home-lab-k3s-argocd` |
| OS/package updates on any host | `home-lab-os-ansible` (planned — ask the operator before creating it) |

A new catalog service usually touches two repos: the one that runs it, plus this repo's catalog table.

## Rules

1. **One fact, one home.** Write a fact only in the repo that owns it (table above); elsewhere link to it.
   Do not copy CIDRs, IP tables, or the catalog into resource repos.
2. **Read-only by default on live systems.** Inspect the router, Proxmox and hosts with read-only commands.
   Any change to live infrastructure needs the operator's explicit approval for that specific change.
3. **Proxmox data comes from the `proxmox` MCP server** (single source of truth for HA, backups, storage,
   networking and guests). Use SSH to hosts only for what the API does not expose (hardware, OS, power).
4. **Public repos.** Never commit secrets, API tokens, WireGuard keys, passwords, the public WAN IP or DDNS
   hostname, serial numbers, or full MAC addresses. Private RFC1918 addresses are fine.
5. **Ask before creating repositories.** New resource repos follow the `home-lab-*` naming.
6. **Docs before code.** The `*-iac` / `*-ansible` / `*-argocd` repos currently document the existing setup;
   don't add IaC code unless the task explicitly asks for it.
7. **Per-repo work runs in its own agent session** started in that repo's directory (Herdr tab,
   `claude --model claude-sonnet-5-5 --effort medium`), coordinated from this repo.

## Access cheat-sheet

| Target | How |
|---|---|
| MikroTik | `ssh bounteous@192.168.11.1` (key auth; read-only by default; see that repo's `AGENTS.md`) |
| Proxmox API | `proxmox` MCP server (node 192.168.11.100, read-only audit token) |
| Proxmox nodes | `ssh root@192.168.11.100` (`panda`), `ssh root@192.168.11.101` (`octopus`) |
| Raspberry Pis | `ssh bounteous@192.168.11.{50,51,52,53}` (`eagle`, `lion`, `penguin`, `shark`) |

SSH keys are provided by the operator's environment; they are never stored in these repos.

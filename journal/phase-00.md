# Phase 0 — Lab setup

Lab 1 ✅ · Lab 2 ✅ · Next: Lab 3 (SSH into the VM with my own key)

## Multipass

```bash
multipass launch --disk 15g --memory 2g --name server   # long flags: -d is disk, -m is memory
multipass list                  # all VMs: state, IP
multipass info server           # details: release, CPU, disk, memory
multipass shell server
multipass stop / start server
multipass delete server         # soft delete — still uses disk, name still taken
multipass recover server        # undo delete
multipass purge                 # permanent, ALL deleted VMs
```

**My VM:** `server` — Ubuntu 26.04.1, 1 CPU, 2 GB RAM (no swap), 15 GB disk, IP `10.149.127.100`

## Virtualization check

```bash
grep -c svm /proc/cpuinfo       # >0 = CPU supports AMD-V (vmx on Intel)
ls -l /dev/kvm                  # exists = kernel KVM driver loaded and usable
```

"Supports" (CPU flag) ≠ "enabled" (BIOS on + driver loaded → `/dev/kvm` exists).

## Quick system checks

```bash
hostname
lsb_release -a                  # Ubuntu release
uname -r                        # running kernel
free -h                         # read "available", not "free"
df -h /                         # root disk usage
ip -br address                  # IPs, one line per interface
uptime                          # time since boot
```

## Laptop network

| Interface | IP | What it is |
| --- | --- | --- |
| `lo` | 127.0.0.1 | loopback |
| `wlp2s0` | 10.97.85.252 | Wi-Fi |
| `docker0` | 172.17.0.1 | Docker bridge (DOWN when no containers) |
| `mpqemubr0` | 10.149.127.1 | Multipass switch = the VM's gateway |
| `tap-…` | — | virtual cable to the VM |

## Facts worth remembering

- **VM has its own kernel.** Same version as the laptop, but different `uptime` proves two kernels. A container would show the *host's* kernel.
- **New kernel runs only after a reboot.** apt installs it on disk; the old one keeps running.
- **VM IP comes from DHCP, keyed on MAC.** Stop/start = same MAC = same IP. Delete + relaunch = new MAC = new IP. Don't hardcode IPs.
- **`10.x` addresses are private** — unreachable from the internet, safe to publish.

## SSH & GitHub

```bash
ssh -T git@github.com           # auth test; -T = no terminal (avoids "PTY allocation request failed")
git config --list --show-origin # every setting + the file it came from
```

- Repo: `git@github.com:zezva802/book-catalog-infra.git` (public) — SSH URL uses my key; HTTPS would need a token.
- Key: `~/.ssh/id_ed25519` (private, must be **600** or SSH refuses it) + `.pub` (on GitHub).
- `known_hosts` = servers I trust (fingerprints). `authorized_keys` = keys allowed to log in **to this machine** (mine is empty).
- Git config: `/etc/gitconfig` (system) < `~/.gitconfig` (user) < `.git/config` (repo) — most specific wins.
- `git status` "ahead/behind" compares with the **saved** `origin/main`, no network. Run `git fetch` for the real picture.
- Never commit: private keys, tokens, passwords, `.env`, connection strings. Deleting later doesn't remove them from history.

## Open

- [ ] Find the raw files behind `free` and `lsb_release`
- [ ] Draw laptop ↔ VM network map from memory
- [ ] Delete `~/dev/book-catalog-infra-draft`

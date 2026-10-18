# Phase 0 — Lab setup

✅ **Phase 0 complete (Oct 18)** — all 4 labs + "Done when" test passed without notes

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

**My VM:** `server` — Ubuntu 26.04.1, 1 CPU, 2 GB RAM (no swap), 15 GB disk, IP `10.149.127.210` (changes every time I recreate it — check with `multipass list`)

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
| `wlp2s0` | changes (DHCP): 10.97.85.252 → 10.253.106.252 | Wi-Fi |
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

## SSH into the VM (Lab 3)

```bash
ssh ubuntu@10.149.127.210       # default Multipass user is ubuntu
# setup, inside `multipass shell server`:
echo "<my .pub line>" >> ~/.ssh/authorized_keys   # >> appends. > would wipe Multipass's key
ssh-keygen -R 10.149.127.100    # remove a stale host from known_hosts
```

| | Laptop (client) | VM (server) |
| --- | --- | --- |
| Private key | `~/.ssh/id_ed25519` — never leaves | — |
| My public key | `~/.ssh/id_ed25519.pub` | copy in `~/.ssh/authorized_keys` |
| Trust the other side | `known_hosts` (VM's host key) | — |
| Server identity | — | `/etc/ssh/ssh_host_*_key` |

How a login works, once per connection:

1. Agree on a session key → everything encrypted from here.
2. Server signs with its host key → laptop checks `known_hosts` (the fingerprint prompt the first time).
3. Laptop offers my public key → server checks it's in `authorized_keys`.
4. Laptop **signs** session data with the private key → server **verifies** with the public key. Signing, not decrypting.

- Public key alone proves nothing — anyone can get it (`github.com/zezva802.keys`). The signature proves I hold the private key.
- Which error comes from which file:
  - `Permission denied (publickey)` → **user key**: mine isn't in the server's `authorized_keys`.
  - `REMOTE HOST IDENTIFICATION HAS CHANGED` → **host key**: the server at that IP doesn't match my `known_hosts`.
- Recreated VM → empty `authorized_keys` + new host key. Same IP → big warning. New IP → just the first-time prompt, and the old entry goes stale.
- On real servers, check the fingerprint against what the provider shows before typing `yes`.
- SSH doesn't care if an IP is private; routers do. `10.149.127.x` is reachable only from my laptop.

## tmux (Lab 4)

Install with `apt`, not snap (snap sandbox gets in the way of terminal tools).

```bash
tmux new -s lab                 # new named session
tmux ls                         # list sessions
tmux attach -t lab              # reattach
```

Prefix: `Ctrl+b`, release, then:

| Key | Does |
| --- | --- |
| `%` | split left / right |
| `"` | split top / bottom |
| arrows | move between panes |
| `o` | next pane |
| `d` | detach (session keeps running) |
| `s` | pick a session |

- tmux = background server that owns the shells; the window is just a viewer. Close the window → shells survive.
- Survives only on the machine running tmux. For long jobs on a server, run tmux **on the server**.

## Carried over to Phase 1

- [ ] Find the raw files behind `free` and `lsb_release` (fits the `/proc` work in Phase 1)
- [ ] Draw laptop ↔ VM network map from memory

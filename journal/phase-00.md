# Phase 0 — Lab setup

## Lab 1 — Multipass VM (Sep 28–29, 2026)

**Goal:** a throwaway Ubuntu server I can break freely, created, stopped, started, deleted and recreated from the command line.

### What I did

1. Checked the CPU supports hardware virtualization and the kernel can use it.
2. Installed Multipass 1.16.4 as a snap.
3. Launched a VM named `server`: 2 GB RAM, 15 GB disk, default image (Ubuntu 26.04.1 LTS, codename resolute), 1 CPU.
4. Inspected it from outside (`info`) and inside (`shell`), and ran the same system checks on my laptop to compare.
5. Stopped and started it, deleted it, found that relaunching fails, purged it, relaunched it.
6. Updated the package lists from inside the VM to prove it can reach the internet (Docker on the laptop didn't break the VM's networking).

### Commands that mattered

```bash
# Is virtualization possible?
cat /proc/cpuinfo                 # found the svm flag = CPU supports AMD-V (next time: filter instead of cat)
ls -l /dev/kvm                    # exists = kernel KVM driver loaded and usable

# Multipass
sudo snap install multipass
multipass version                 # 1.16.4
multipass list                    # all instances: state, IP, image
multipass launch --disk 15g --memory 2g --name server
multipass info server             # one instance in detail: release, CPU, disk, memory, load
multipass shell server            # open a shell inside the VM
multipass stop server
multipass start server
multipass delete server           # soft delete: moves to "Deleted", name still taken
multipass purge                   # empties the bin for good (ALL deleted instances)

# System checks (ran on VM and laptop)
hostname                          # machine name
lsb_release -a                    # Ubuntu release
uname -r                          # kernel version
free -h                           # memory (read "available", not "free")
df -h /                           # root disk size and usage
ip address                        # interfaces and their IPs (full)
ip -br address                    # same, one line per interface

# Inside the VM
sudo apt update                   # refresh the package catalogue (installs nothing)
```

### VM vs laptop

| Check | VM (`server`) | Laptop (`terga-ubuntu`) |
| --- | --- | --- |
| Hostname | server | terga-ubuntu |
| Release | Ubuntu 26.04.1 LTS | Ubuntu 26.04.1 LTS |
| Kernel | 7.0.0-31-generic | 7.0.0-31-generic |
| Memory total | 1.9Gi, no swap | 9.1Gi, 4.0Gi swap |
| Root disk | 14G | 233G |
| IP | ens3 10.149.127.30, then 10.149.127.100 after recreating | wlp2s0 10.97.85.252 |

Laptop interfaces: `lo` 127.0.0.1 (UNKNOWN), `wlp2s0` 10.97.85.252 (UP, Wi-Fi), `docker0` 172.17.0.1 (DOWN, no containers), `mpqemubr0` 10.149.127.1 (UP, Multipass switch, the VM's gateway), `tap-2042d79a31d` (UP, virtual cable, no IPv4).

### What broke and why

- **Swapped flags.** First wrote `multipass launch -n server -d 2g -m 15g`: that asks for a 2 GB disk and 15 GB RAM. `-d` is disk, `-m` is memory. Fix: use long flags (`--disk`, `--memory`) so the command explains itself.
- **Relaunch failed:** `launch failed: instance "server" already exists`. `delete` is a soft delete: the VM sits in a recycle bin, still takes disk space, and keeps its name. `purge` removes it for good. `recover` brings it back.
- **`ip -br` printed the usage text.** The usage line is `ip [ OPTIONS ] OBJECT { COMMAND | help }`: `[ ]` is optional, no brackets is required, `{ a | b }` means pick one. I gave an option but no object. Fix: `ip -br address`.
- **Guessed command names** (`sysinfo`, `unix -v`, `memory`) → "command not found". Fix: search the man pages by keyword instead of guessing.
- **The IP changed** from .30 to .100 after recreating the VM, so anything that stored the old IP would break.

### Concepts — in my own words

_Answer each without scrolling up. If I can't, that's what to review._

- VM vs container. Why do the VM and laptop show the same kernel version, and what would `uname -r` show inside a container?
  -
- What is `/dev`? What's the difference between `/dev`, `/proc` and `/sys`?
  -
- The 5 links in the KVM chain, and what a missing `/dev/kvm` tells me:
  -
- `free` vs `available` in `free -h`, and what real memory trouble looks like:
  -
- Why does my laptop have four different IPs?
  -
- IP address vs port:
  -
- What `/24` means, and what a gateway is:
  -
- Why the VM's IP changed, and three ways to stop depending on an IP:
  -
- `delete` vs `purge`, and why tools make deleting a two-step action:
  -

### Quiz

About 2.5 / 5. Solid on free vs available and delete vs purge. Half-right on VM vs container kernel and the KVM chain: I had the idea but not the precise distinction ("supports" vs "enabled", "OS files" vs "kernel"). Didn't recall DHCP and why IPs change: review before Lab 3.

### Unfinished

- [ ] Find the raw files `free` and `lsb_release` read from, and match them to the command output.
- [ ] Prove the VM and laptop run separate kernels even though the versions match (hint: boot time).
- [ ] Draw the laptop ↔ VM network map from memory in the notebook.

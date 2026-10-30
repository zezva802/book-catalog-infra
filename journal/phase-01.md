## Lab 1 — shared folder /srv/app
- /srv/app: root:devs, mode 2770 (setgid + rwx owner/group, nothing for others)
- setgid on a dir → new files get the dir's group, not the creator's
- umask removes bits from new files (666 files / 777 dirs). 002 → 664, 022 → 644
- sudo applies its own umask 022 → test as a real login (su - user / sudo -iu user)
- usermod -aG: ALWAYS -a, or -G replaces all groups (sudo lockout)
- group changes apply at next login, not in the current session
- "Permission denied" = rwx bits; "Operation not permitted" = not the owner (chmod/chown)
- dir bits: r = list, w = create/delete/rename, x = enter

## Lab 2 — top 5 processes by memory
- ps aux | sort -k 6,6nr | head -n 5
- sort -k 6,6nr: column 6 only (RSS), numeric, biggest first. -k 6 alone = col 6 to end of line
- process = running program: PID, PPID, UID/GID, own memory, open files
- /proc = virtual fs, kernel builds text on read → size 0 in ls, but cat works
- /proc/<pid>/ = per process; /proc/cmdline etc. = system-wide (cmdline there = kernel boot params)
- $$ = current shell's PID → /proc/$$/status, /proc/$$/cmdline
- -bash (dash) = login shell; cmdline args are NUL-separated
- grep '^Vm' file (no cat needed; ^ = start of line)
- VSZ/VmSize = reserved address space (mostly not in RAM); RSS/VmRSS = actually in RAM → "memory used"
- ps units = KiB; %MEM = RSS / total RAM; ps just reads /proc
- RSS counts shared libs per process → sum of RSS > total RAM
- [brackets] + 0 memory = kernel threads
- environ (400), mem (600), vmallocinfo (root) = private: secrets / kernel addresses
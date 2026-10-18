## Lab 1 — shared folder /srv/app
- /srv/app: root:devs, mode 2770 (setgid + rwx owner/group, nothing for others)
- setgid on a dir → new files get the dir's group, not the creator's
- umask removes bits from new files (666 files / 777 dirs). 002 → 664, 022 → 644
- sudo applies its own umask 022 → test as a real login (su - user / sudo -iu user)
- usermod -aG: ALWAYS -a, or -G replaces all groups (sudo lockout)
- group changes apply at next login, not in the current session
- "Permission denied" = rwx bits; "Operation not permitted" = not the owner (chmod/chown)
- dir bits: r = list, w = create/delete/rename, x = enter
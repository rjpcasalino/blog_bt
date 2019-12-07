---
title: 'Arch: Getting Started'
layout: post
---

# Before

Validate the install ISO! I think I do this half-way so my install might have some Chinese malware maybe? I downloaded from a server that purported to be in Arizona.

# After

Having installed Arch a few times and never having as simple/joyful a time as I would like I made this. 

"This" being this very document you see before you. It should complement the official guide.

First, use `parted` and pass the values and reference install guide for partition layout suggestions.

After a install and reboot I am sometimes greeted with a `connect: Network is unreachable` on a `ping 8.8.8.8`.

If, for whatever reason, you don't have access to `dhcpcd` package do this (static IP):

```
$ ip link set interface up // if down
$ ip address show 
$ ip address add <address/prefix_len> broadcast + dev <interface>
$ ip route add default via 192.168.0.x // default gateway 
```

Ensure that /etc/resolve.conf has been created and that it's setup properly...

e.g.,:

```
nameserver 8.8.8.8 // google
```


otherwise,

```
$ dhcpcd <interface> 
```

Ensure the services are enabled to start at boot:

```
$ systemctl enable dhcpcd.service
```

### General Tips

In no particular order after an install:

- `useradd -m xxx` (?)
- `pacman -S openssh`
- `pacman -S git` 

AUR (Arch User Repository); [yay](https://github.com/Jguer/yay) wraps it with go. Anyway, use AUR packages at own risk. Replaces Yogurt.



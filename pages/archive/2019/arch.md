---
title: 'Arch: Getting Started'
layout: post
---

I've installed Arch a few times and it's never been as simple as I would hope. Hopefully, this document should complement the official guide.

Often I am greeted with a `connect: Network is unreachable` on a `ping 8.8.8.8`.

Use `parted` and pass the values and reference doc for layout.

If, for whatever reason, you don't have access to `dhcpcd` package: 

```
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




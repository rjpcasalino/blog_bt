---
title: 'Flask, Guniorn, and Unix Sockets'
layout: post
---

In this post I will show one how to get a Flask deployed in NixOS using a varity of tools including: nginx, gunicorn, unix sockets, and nix.

First and foremost, let's make sure NixOS in on 19.09 which is the version our server is running. 

```
$ how to update nix channels and etc etc
```

Next, remind yourself of the various nginx options via their own [docs](url here) and also how to properly represent those options using the approoaoate nix syntax. Nix nginx options can be found [here](url here).

```
$ example of basic nixos nginx config
```

Ensure that simp_le (simple let's encrypt) is above version `0.9.0` as ACMEv1 support is being depercated by Let's Encrypt. 

Our stepup in the end should look thus:

- nginx on chud
- routes to
- nginx on Pi4
- routes to
- Unix socket
- gunicorn running as a service via systemd


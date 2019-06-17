---
title: nix
layout: post
---

I moved off EC2 and back to bare metal. On an old Samsung 1TB drive from 2008 this HTML file sits. How fast the Nix addiction spreads! 
I am running NixOS upon all my machines. Such are the actions of madmen! Well, that's not actually entirely true. I am not running Nix 
on my desktop box. Its anointment will come later as I get more comfortable cruising on my ThinkPad. NixOS has been such a joy. Ease 
of installation is a key indicator of a worthwhile distribution. Nix has that. I plopped down beside the makeshift workstation I had 
assembled in my office and surfed the information superhighway to the NixOS manual: 

(Imagine, if you please, an American sitting on his home office sofa besides his cat.)
If imagination is not your bag, you can always scope out the [image](static/asset/path).

### Getting started

*UEFI*

> parted /dev/sda -- mklabel gpt

> parted /dev/sda -- mkpart primary 512MiB -8GiB

> parted /dev/sda -- mkpart primary linux-swap -8GiB 100%

> ** Note: The swap partition size rules are no different than for other Linux distributions. **

> parted /dev/sda -- mkpart ESP fat32 1MiB 512MiB

> parted /dev/sda -- set 3 boot on

**For legacy boot (remember that this can all be found in the manual as well!)**

> parted /dev/sda -- mklabel msdos

> parted /dev/sda -- mkpart primary 1MiB -8GiB

> parted /dev/sda -- mkpart primary linux-swap -8GiB 100%

*Format it*

> mkfs.ext4 -L nixos /dev/sda1

> mkswap -L swap /dev/sda2

* for UEFI systems

> mkfs.fat -F 32 -n boot /dev/sda3

*mount it*

> mount /dev/disk/by-label/nixos /mnt

*for UEFI*

> mkdir -p /mnt/boot

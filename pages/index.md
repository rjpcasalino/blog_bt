---
title: Boring Tranquility
layout: basic
---

_There is an ongoing -- albeit abated -- stream, which is all about its flow. We can learn much from the play of water upon the 
shore. Just seeing a gentle caress spreading liquid via the so numerous infinitesimal canyons of sand is enough. - Bernard Truman_

["Liberty and Union, now and forever, one and inseparable!"](https://en.wikipedia.org/wiki/Daniel_Webster)


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

> mount /dev/disk/by-label/boot /mnt/boot

*gen the config*

> nixos-generate-config --root /mnt

Edit the file using nano (make sure you are editing the mounted config)

**do the install**

> nixos-install

Look to a commands man page to explain anything (e.g., flags) that you don't understand. Or, dare I say it, just Google it.

|~~~|
    
Odyssey continues its roll to release 0.0.1. Odyssey is a functional static site generator that allows templates to be written using 
[jinja](http://jinja.pocoo.org/). It isn't powerful, it isn't that smart, but it works in what it is designed to do today. Odyssey was 
started with a desire to better understand Python. It's been a fun time thus far, so I plan to keep going. My true end goal is to have 
a simple, functional command line tool that allows a user to use the `odyssey start` cmd to spin up a https website (i.e., get certs 
from Let's Encrypt, spin up nginx). We'll see how far I can get. Odyssey gets its name from Home's 
[Odyssey](https://open.spotify.com/album/2Nz9gdj35Unk1AbfL8Igmx?si=WhfO0FV1S4GC66_5GqhyxQ) and the idea that Odyssey attempts to take 
you on a splendid trip to places unknown.

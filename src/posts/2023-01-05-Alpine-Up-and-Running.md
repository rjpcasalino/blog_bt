---
title: "Using Alpine on ASUS VivoBook E203MAS_L203MA 1.0"
Layout: post
---

> I said dance, not fuck. Don't get it confused. — Cardi B

Whereas there are offical docs and myraid guides to follow regarding setting up Alpine Linux I decided to record the process on a brand newASUS VivoBook. Why the E203 VivoBook? I'm a big fan of the ASUS EeePC series and the E203 is a worthy successor. Simple and small, lightweight, and with a keyboard that pleases the fingertips: the E203 checked all my boxes.

First, use [etcher](https://www.balena.io/etcher/) to create a bootable USB drive.

<hr>

Second, turn off secure boot in the BIOS (get to the BIOS with F2 on this box).

Third, this is a simple sys install (not running from RAM) so just follow steps given in installer.

Fourth, you will need the community repos so edit `/etc/apk/repositories` and remove the comment to allow downloading from community repos.

Finally, as root begin installing goodies:

	# become root
	su -
	# add basics
	apk add man mandocs openbox xterm terminus-font doas

...and some other crap (using doas):

	apk add git neofetch feh # feh is for wallpaper

	apk add cwm # calm window manager

	apk add xinit

	apk add light # for backlight control

	apk add dbus dbus-openrc # for bluetooth

	apk add bluez bluez-openrc bluez-alsa

	# bluez-alsa is needed to get bluetooth headsets to work
	# oddly found this here: https://wiki.postmarketos.org/index.php?title=Bluetooth&mobileaction=toggle_view_desktop
	# Getting bluetooth working requires dbus and bluez 
	# Check out dbus here: https://www.freedesktop.org/wiki/Software/dbus/
	# Remember to add bluetooth to runtime so it works at boot
	# audio is easy with alsa drivers and such

	apk add alsa-utils alsa-utils-doc alsa-lib alsaconf alsa-ucm-conf

	# read more about alsa here: https://en.wikipedia.org/wiki/Advanced_Linux_Sound_Architecture
	# to change shell we need libuser

	apk add libuser

	# Alpine uses ash as default shell but we want bash
	apk add bash

	# Ok, done with apk for now
	touch /etc/login.defs
	# make sure /etc/default is around
	touch /etc/default/adduser

	# pick bash
	doas lchsh <user>

	rc-update add dbus default
	rc-update add bluetooth
	rc-update add bluealsa
	setup-xorg-base # do magic
	addgroup <user> audio
	addgroup <user> audio
	addgroup <user> input
	addgroup <user> video

	# logout so the changes take effect
	# startx needs xinitrc to load the Xresources file
	# and start cwm
	# Either run install.sh from Shangri-la or do whateve u like
	touch .xinitrc 
	# or copy contents (TODO make install.sh do that)
    # before starting X, install setxkbmap for compose key
    apk add setxkbmap
    # see .XCompose for more
    # then, startx!
	startx

Ok, once that stuff is out of the way let's work on the touchpad. Install and read about xinput:

	apk add xinput xinput-doc

In order to discover the key mapping of the touchpad issue this command:

	xinput list
And from there once you've discovered what device, get the button map via:
	xinput get-button-map "<ur device>"
And when you are ready, set the button map with:
	xinput set-button-map 1 2 3 4 5 6 7

The mapping is in physical order and setting one of the above values with "0" will disable the button. Thus setting 2 to "0" will disable that action.

To get suspend to work on LID close follow this guide: https://wiki.alpinelinux.org/wiki/Suspend_on_LID_close
You might need to create the directories and don't forget to make the script you'll copy executable.

And here are the fonts that work best for most setups:
	apk add terminus-font ttf-inconsolata ttf-dejavu font-noto font-noto-cjk ttf-font-awesome font-noto-extra

### Blogging and nix flakes and docker
Before starting let's install docker:

	apk add docker
	addgroup <user> docker

I use a static site generator of my own (less than steller) making called bss — here's how to get it working with Nix and Docker:

	# Once you've cloned it, mount the dirs and expose the ports
	$ docker run -it --rm -v $(pwd)/bss/:/bss -v $(pwd)/blog_bt:/blog_bt -p 8000:8000 nixos/nix
	# bss is flakes based so we have to pass annoying flags but this can also be set in nix config
	$ nix build --extra-experimental-features nix-command --extra-experimental-features flakes
	# copy the resulting program into a dir on the PATH
	$ cp result/bin/bss  /nix/var/nix/profiles/default/bin/

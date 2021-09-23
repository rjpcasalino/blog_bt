---
title: 'Using the calm window manager'
layout: post
---

The calm window manager (cwm) is a "lightweight and efficient window manager for X11" written mostly by [Marius Eriksen](https://monkey.org/~marius/). It purports to "maintain the simplest and most pleasant aesthetic."

The control keys are fairly simple (ctrl, meta, shift):

	CM-Return	Spawn a new terminal.
	...
	M-Return	Hide current window
	...
	M-question	Spawn “exec program” dialog.
	M-period	Spawn “ssh to” dialog.

There are 35 odd-ish key bindings. That's not too bad. cwm's "GROUPS" concept makes it easy to simulate "virtual desktops". Groupings can be traversed via arrow keys. 

One can move windows about with key bindings one would find in vim (i.e., hjkl).

Configuration is pleasant and straightforward (*.cwmrc*):

	command firefox  firefox
	command shot	 scrot
	command pad 	 "xterm -e ed /home/rjpcasalino/yellowpad.md"

	borderwidth 1
	# Inherit groups 
	sticky yes

	# Define a “gap” in pixels at the edge of the screen. Maximized windows will not overlap gap area.
	gap 24 0 0 0
	# Ignore programs by name, not drawing borders around them.
	ignore sbar_ip
	ignore sbar_time

	# miss i3? Snapping made easy:
	bind-key CM-Right		window-snap-right
	...
	bind-key XF86MonBrightnessUp	"light -A 10"
	bind-key XF86MonBrightnessDown	"light -U 10"

Commands can be called with C-question and allow one to open applications quickly. Likewise, M-period opens an ssh session which reads your .ssh/config. 

Ersatz tiling can be achieved with key-bindings for window snapping. 

### Using cwm with NixOS

[Using X without a Display Manager](https://nixos.wiki/wiki/Using_X_without_a_Display_Manager) on NixOS isn't intuitive but easy to do nevertheless.

Here's my configuration XServer block:

	  # Xserver 
	   services.xserver = {
	    enable = true;
	    # We don't want to autorun X; we want startx 
	    autorun = false;
	    # Whether to symlink the X server configuration under /etc/X11/xorg.conf. 
	    exportConfiguration = true;
	    displayManager.startx.enable = true;
	    windowManager.cwm.enable = true;
	  };

Ryan Chan has a nice [blog post](https://rycwo.xyz/2019/02/07/nixos-series-configuring-xinit) that was helpful.

The contents of `.xinitrc`:

	#!/usr/bin/env sh

	xrdb -load $HOME/.Xresources

	xterm -cr "#fff" -g 13x1+30+0   -hold -e sbar_ip &
	xterm -cr "#fff" -g 17x1-0+0    -hold -e sbar_time &
	xterm -cr "#fff" -g 4x1+0+0     -hold -e sbar_bat &

	$HOME/.fehbg &

	exec cwm 

<!-- ![cwm screenshot](https://wiki.boringtranquility.io/assets/imgs/cwm_grab.png) -->

- - -

Here's some links that are helpful if you're interested in cwm:

* [cwm(1)](https://man.openbsd.org/cwm.1)
* [cwmrc(5)](https://man.openbsd.org/cwmrc.5)
* [Getting started with cwm](https://undeadly.org/cgi?action=article&sid=20090502141551)
* [Netbooting OpenBSD](/posts/2021-08-19-Netbooting-OpenBSD.html)

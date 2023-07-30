---
title: 'Using the calm window manager'
layout: post
---

The calm window manager (cwm) is a "lightweight and efficient window manager for X11" written mostly by [Marius Eriksen](https://monkey.org/~marius/). It purports to "maintain the simplest and most pleasant aesthetic."

The control keys are fairly simple (CTRL, META, and SHIFT):

    CM-Return	Spawn a new terminal.
    ...
    M-Return	Hide current window
    ...
    M-Question (?)	Spawn "exec program" dialog.
    M-Period (.)	Spawn "ssh to" dialog.

There are 35 odd-ish key bindings. That's not too bad. cwm's "GROUPS" concept makes it easy to simulate "virtual desktops". Groupings can be traversed via arrow keys.

One can move windows about with key bindings one would use if one were familiar with vim (i.e., h j k l).

Configuration is pleasant and straightforward (*.cwmrc*):

    # Set fonts
    fontname "ProFontWindows Nerd Font Mono:size=13"

    # Overide the default (xterm)
    command term    "kitty"
    # more examples
    command firefox "firefox"
    command shot    "scrot"

    # geometry stuff
    borderwidth 1
    # Inherit groups
    sticky yes

    # Define a "gap" in pixels at the edge of the screen. Maximized windows will not overlap gap area.
    gap 24 0 0 0

    # Ignore programs by name, not drawing borders around them.
    ignore polybar

    # Miss i3? Snapping made easy:
    bind-key CM-Right		window-snap-right
    # More examples
    bind-key XF86MonBrightnessUp	"light -A 10"
    bind-key XF86MonBrightnessDown	"light -U 10"

Applications listed in the config such as firefox can be selected with C-Question, which allow one to open them quickly. As mentioned, M-Period will spawn an ssh session which reads one's .ssh/config. Use M-Question if the command isn't listed in your config. Recall that this opens the "exec program" dialog.

Ersatz tiling can be achieved with key-bindings for window snapping (see above).

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

Ryan Chan has written an excellent [blog post](https://rycwo.dev/archive/nixos-series-004-configuring-xinit/) that was invaluable. Many thanks to him.

The contents of `.xinitrc`:

	#!/usr/bin/env sh

	xrdb -load $HOME/.Xresources

	setxkbmap -option compose:ralt

	polybar &
	xscreensaver &

	exec cwm

<!-- ![cwm screenshot](https://wiki.boringtranquility.io/assets/imgs/cwm_grab.png) -->

- - -

Here are helpful links for those interested in cwm and more:

* [cwm(1)](https://man.openbsd.org/cwm.1)
* [cwmrc(5)](https://man.openbsd.org/cwmrc.5)
* [Getting started with cwm](https://undeadly.org/cgi?action=article&sid=20090502141551)
* [Netbooting OpenBSD](/posts/2021-08-19-Netbooting-OpenBSD.html) - cwm is an official window manager for OpenBSD and works well there too. Reading this one will learn more than one cares to regarding OpenBSD and net booting basics. Getting the cwm running with X forwarding is not covered as yet.
* [Some "hidden" or esoteric options can be found here - thanks Joel](https://www.tumfatig.net/2021/from-clean-green-mockup-to-openbsd-cwm1-desktop/)

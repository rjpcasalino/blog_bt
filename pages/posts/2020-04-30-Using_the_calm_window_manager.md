---
title: 'Using the calm window manager'
layout: post
---

The Calm Window Manager (cwm) is a "lightweight and efficient window manager for X11" written mostly by [Marius Eriksen](https://monkey.org/~marius/). It preports to "maintain the simplest and most pleasant aesthetic." Having used it for nearly two weeks it's hard to disagree. 

The control keys are fairly simple:

```bash
           CCMM--RReettuurrnn       Spawn a new terminal.
           CCMM--DDeelleettee       Lock the screen.
           MM--RReettuurrnn        Hide current window.
           MM--DDoowwnn          Lower current window.
           MM--UUpp            Raise current window.
	   ...
           MM--qquueessttiioonn      Spawn “exec program” dialog.
           MM--ppeerriioodd        Spawn “ssh to” dialog.  This parses _$_H_O_M_E_/_._s_s_h_/_k_n_o_w_n___h_o_s_t_s to provide host auto-completion.  ssh(1) will be executed via the configured terminal emulator.
           CCMMSS--rr           Restart.
```

35 odd-ish commands. That's not too bad. cwm's "GROUPS" concept makes it easy to simulate "virtual desktops". Groupings can be traversed via C-arrow keys. 

Moving windows utitlzies vim keys(i.e., HJKL).

Configuration is pleasant and straightforward:

```bash
command firefox  firefox
command shot	 scrot
command pad 	 "xterm -e ed /home/rjpcasalino/yellowpad.md"

borderwidth 2
sticky yes

gap 24 0 0 0
ignore showip
ignore showtime

bind-key XF86MonBrightnessUp	"light -A 10"
bind-key XF86MonBrightnessDown	"light -U 10"
```


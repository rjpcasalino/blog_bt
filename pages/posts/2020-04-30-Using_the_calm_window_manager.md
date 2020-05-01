---
title: 'Using the calm window manager'
layout: post
---

The Calm Window Manager (cwm) is a "lightweight and efficient window manager for X11" written mostly by [Marius Eriksen](https://monkey.org/~marius/). It preports to "maintain the simplest and most pleasant aesthetic." Having used it for nearly two weeks it's hard to disagree. 

The control keys are fairly simple (ctrl, meta, shift):

```bash
           CM-Return	Spawn a new terminal.
           CM-Delete    Lock the screen.
           M-Return     Hide current window.
           M-Down       Lower current window.
           M-Up		Raise current window.
	   ...
           M-question	Spawn “exec program” dialog.
           M-period	Spawn “ssh to” dialog.
           CMS-r        Restart.
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


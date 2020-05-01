---
title: 'Using the calm window manager'
layout: post
---

The calm window manager (cwm) is a "lightweight and efficient window manager for X11" written mostly by [Marius Eriksen](https://monkey.org/~marius/). It purports to "maintain the simplest and most pleasant aesthetic." Having used it for nearly two weeks it's hard to disagree. 

The control keys are fairly simple (ctrl, meta, shift):

```bash
           CM-Return	Spawn a new terminal.
	   ...
           M-Return     Hide current window.
	   ...
           M-question		Spawn “exec program” dialog.
           M-period		Spawn “ssh to” dialog.
```
and so on...

There are 35 odd-ish key bindings. That's not too bad. cwm's "GROUPS" concept makes it easy to simulate "virtual desktops". Groupings can be traversed via arrow keys. 

Moving windows utilizes vim keys(i.e., hjkl).

Configuration is pleasant and straightforward (.cwmrc):

```bash
command firefox  firefox
command shot	 scrot
command pad 	 "xterm -e ed /home/rjpcasalino/yellowpad.md"

borderwidth 1
sticky yes

gap 24 0 0 0
ignore showip
ignore showtime

bind-key XF86MonBrightnessUp	"light -A 10"
bind-key XF86MonBrightnessDown	"light -U 10"
```

Here's some links that are helpful if you're interested in cwm:

* [cwmrc(1)](https://man.openbsd.org/cwm.1)
* [cwmrc(5)](https://man.openbsd.org/cwmrc.5)
* [Getting started with cwm](https://undeadly.org/cgi?action=article&sid=20090502141551)







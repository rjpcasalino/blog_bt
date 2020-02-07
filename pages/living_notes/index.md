---
title: 'Living Notes'
layout: post
---

*A living document of notes related to tech*

- `pgrep` is a useful tool for finding a process in Unix-like systems (i.e., Linux)
	- `pgrep -l firefox` will print the PID as well as the process name
- `kill -9 <PID>` force kill a process 
- Use `<span>` and &mdash to create an EM DASH: <span>&mdash;</span> 
	- MDN has a great reference concerning HTML entities (https://developer.mozilla.org/en-US/docs/Glossary/Entity)
	- > An HTML entity is a piece of text ("string") that begins with an ampersand (&) and ends with a semicolon (;) . Entities are frequently used to display reserved characters (which would otherwise be interpreted as HTML code), and invisible characters (like non-breaking spaces). You can also use them in place of other characters that are difficult to type with a standard keyboard.  
- Better understanding of [GnuPG](https://gnupg.org/faq/gnupg-faq.html) is probably a prerequisite for any programmer approaching the five year mark.
- Read up on [Varnish](https://varnish-cache.org/docs/6.3/index.html) 
- `rfkill` - tool for enabling and disabling wireless devices; sometimes my bluetooth controller will tell me there is no default controller found or whatever error it decides to spit out. `rfkill` can sometimes help with those issues. I will need to read up on the diff between soft and hard blocks.
- `setlocal` in `.vimrc`: [stackexchange answer](https://vi.stackexchange.com/questions/14605/differences-between-local-global)
- [CSS Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout) is pretty amazing.
	- [grid-template-areas](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-areas) seem particularly fun.

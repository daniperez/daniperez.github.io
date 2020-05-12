---
layout: post
title: "Rapoo T300P touch-pad in Linux"
date: 2016-03-04 08:00:00 +0200
# categories: 
tags: [gnome, linux, fedora, mouse, usb, gnome, shell, touchpad, mice, touch-pad, rapoo, t300p]
---

I'm sharing this since it doesn't seem to exist much information about it in The Internets.

I replaced my mouse [by a touch-pad by Rapoo](http://rapoo.com/ProductShow.aspx?PType=8fQpYH%2b%2b7i8%3d&PID=3P9f7SY0Eec%3d) and I was glad to confirm that it works in Linux. At least in my Fedora 19 installation with Gnome.

The following gestures, as found in Rapoo's manual, work as follows with no configuration whatsoever:

*   One-finger touch: left click.
*   One-finger swipe: moves the pointer.
*   One-finger swipe from the edge: ~~supposedly switches applications~~. **Does not do anything**.
*   Two-finger touch: right click.
*   Two-finger swipe: scroll up/down left/right.
*   Three-finger swipe: swiping up has the same effect as pressing the Windows key in Gnome Shell. Swiping left and right goes back and forward respectively (e.g. in a browser). **Swiping down doesn't do anything**.
*   Two-finger touch: middle click.
*   Four-finger swipe: swiping down shows Gnome Shell's task bar (same as *Windows+M*). **Swiping down doesn't do anything**.
*   Two-finger zoom: zooms in and out as expected.

In summary, it's fully working. The missing functions are probably events from the touch-pad not bound by default in Gnome. I didn't try to bind them since the defaults are enough for me. Not only it's working but also the gadget is small and very stylish. The touch-pad is cordless and has a battery that can be charged with a standard USB cable, furnished in the package as well as a tiny USB 5G (sic) wireless receiver. I liked also that the package box was very small as well (i.e. less waste).

Good work Rapoo and Gnome / Dbus / Fedora or whoever is responsible for this "plug&play" experience!

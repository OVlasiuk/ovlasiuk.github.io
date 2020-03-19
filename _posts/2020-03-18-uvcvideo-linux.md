---
title:  Usable skypeforlinux setup with some rant-shaped backstory
date:   2020-03-18 08:00:00 -0500
layout: single
categories: Linux config

---

## TL;DR

The following [setup](#a-somewhat-usable-setup) allows to place video calls with Skype on a Linux (Manjaro) without having the webcam to crash very often. The gory details are in [the backstory](#the-backstory). In general, it is wise to avoid reading rants on the internet.

## A somewhat usable setup
We create a symlink to the functional video device by a `udev` rule, and use the symlink in all apps. Mysteriously it leads to fewer problems with video in calls. [udev](https://wiki.archlinux.org/index.php/Udev) is a part of systemd.

Create the file `/etc/udev/rules.d/40-webcam.rules`. The number 40 corresponds
to the priority of the rule. It is not important unless you have put some other
rules in `rules.d` before, in which case you probably know what to do, and should not
be reading this. In the file, add 
```
SUBSYSTEMS=="usb", ATTR{index}=="0", ATTR{name}=="Integrated Camera: Integrated C", SYMLINK+="video9"
```
replacing the above attributes `ATTR{...}` with the attributes of your camera. Run 
```
$ sudo udevadm trigger
```
to force udev to apply the changes. Verify that the symlink has been created by `ls /dev/video9`.  Then, in every relevant app select the *second* option for the camera. The first will
likely be one of `video0` and `video1`. The second will be the `video9` device,
which will link to the functional device among `/dev/video*` due to the udev rule.

To determine your attributes, do the following.
If you have a single camera, the functional (non-metadata) interface is likely
to have the index equal to 0, as shown above, so `ATTR{index}=="0"` will work. To verify this, inspect your video
devices by
```
$ ls /dev/video*
```
Then, determine the useful one, and the one with metadata by comparing the
outputs of 
```
$ v4l2-ctl --device=/dev/videoN --all 
```
for different `N`.
The actual video device will have physical properties: frequencies, exposure, fps, etc.
To pick the attributes that can used in the udev rule above, see the output of 
```
$ udevadm info --attribute-walk --name=/dev/videoN
``` 

## The backstory

Placing Skype video calls on Linux has been really unpleasant the last couple of times I
had to do it. The video would crash every 30 to 90 seconds, and the only way to
get it up would be to disable and enable it back. Then, it would crash again.
"Skype is still really bad on Linux", I said to myself. "Zoom does not have this
issue!", I thought. To make sure this was a purely Skype bug, I tested the
camera using guvcview, a native gtk app. Then, the video stream in guvcview froze
in 30 seconds after it was started. Oh-oh.

My first inkling was to look at the pair of video devices under `/dev/`, `video0`
and `video1`. At an earlier point I was already marveling at the
engineering ingenuity of creating two logical devices for a single camera. Only one of them was functional, often called `video0`. Well,
perhaps Skype is confused by the other, non-functional `device1`? According to
a number of online discussions, this `device1` was carrying some extremely
important but apparently useless for my purposes metadata. Hmm, I should be able to
disable it then. Unfortunately, I was not — not even sure where this would be done; perhaps also as a `udev` rule.
Curiously, it became apparent that the introduction of this metadata channel circa kernel 4.15 has been accompanied by
declaring the userland apps as being responsible for getting it all straight [[1]][[2]]. In fact,
the kernel was doing everything right by presenting two nearly-identical devices,
and the userland apps had to adjust to the everchanging kernel interfaces. The
kernel has no compatibility bugs, only features [[3]]. This
reasoning felt vaguely familiar. *I bet Linus would not approve of it*. Wait,
I remember now — I have read this *exact* argument Linus was having... with this
*exact* person defending uvcvideo, here [[4]]?  Oh-oh.

Well, to be fair, Skype has in fact adapted to the everchanging kernel behavior. It used to be
confused by the two different camera devices long time ago, closer to version 4, but any even slightly recent version was
correctly seeing only one camera. This seemed to be the problem of my particular
setup.

The logical step was to run kernel 4.14, to see if that mitigates the issue.
Aaand, under 4.14 there was only `/dev/video0`, so that certainly looked
promising... except that guvcview crashed just as gloriously in 60 seconds, and
skypeforlinux refused to start at all. Which made the entire exercise even more
futile. Further investigation of these crashes revealed that the
numbers of `/dev/video*` files were constantly changing. According to `dmesg`,
the internal camera of my laptop appeared to (re)connect sporadically. Such
reconnections caused the video device number to be reassigned, which made the
whatever video app used it to crash. Still, likely something even more
nefarious was going on here, as *Zoom did not seem to experience any of this?*

At this point the *somewhat usable setup* was found, and rid me of the sad
necessity of poking around Linux internals (for now).

[1]: https://bugzilla.kernel.org/show_bug.cgi?id=199575
[2]: https://unix.stackexchange.com/a/539573
[3]: https://bugzilla.kernel.org/show_bug.cgi?id=199575#c2
[4]: https://lkml.org/lkml/2012/12/23/75

## Disclaimer 
Nobody owes me (or anyone else) to develop open source software. I am grateful
that things work already the way they do. [Eventually](https://xkcd.com/644/), everything is going to be
supported by the kernel.



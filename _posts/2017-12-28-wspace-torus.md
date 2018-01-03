---
title:  "Workspace grid for Cinnamon"
date:   2017-12-27 08:00:00 -0500
layout: singlemath
categories: Code Python

---

I usually work with *many* windows open at the same time. Say, writing something
in $$\LaTeX$$ means having editor and a couple of PDF-viewer windows open; throw
in browser, reference manager, a terminal or two, and you easily end up at 8-10
windows. Unfortunately, switching between as many as 5 on a single workspace is
already rather tedious, be it by clicking and dragging, or by `Alt-Tab`bing your
way around. One would think that a good window manager, [Cinnamon][1] being
an example of such, should allow to fix this. 

For some reason though, Cinnamon only provides left/right inter-workspace motions out of
the box; I find it more natural to think of the workspaces as sitting on a 2d grid. Apparently, there *are* people sharing this sentiment: a quick search reveals a [discussion][SE] on unix stackexchange, where somebody mentions using 16(!) workspaces. Besides, there already is an applet that purports to do precisely what I want; the fact that it conflicts with Expo and does not allow to move windows around makes it somewhat suboptimal.

The ~~suitable solution~~ hack was to write a basic wrapper around `wmctrl` in
Python and to tie shortcuts to it using the Cinnamon config. There is also
`xdotool`, which would probably do the trick as well. 

Enabling the "grid" option in Expo further means being able to get an overview of all
the windows like so:

![Expo view]({{ "/assets/images/Expo9.png" | absolute_url}})

I must add that my default config only includes 4 workspaces; nothing outrageous like shown here. The
`wmctrl` allows to both jump between workspaces and move windows around: 

> wspacetorus.py -d up

> wspacetorus.py -t left

etc., with "-d" for "desktop" and "-t" for "to (desktop)" performing jumping and moving the active window in the specified direction. The script is below and in the [repo](https://github.com/OVlasiuk/snippets). I imagine it would run on any WM compatible with `wmctrl` so, according to the manpage, any "EWMH/NetWM compatible X Window Manager".



[1]: https://en.wikipedia.org/wiki/Cinnamon_(software)
[SE]: https://unix.stackexchange.com/questions/127897/cinnamon-2-2-how-to-add-custom-shortuct-move-workspace-up


{% highlight python linenos %}
{% raw %}
#!/usr/bin/env python

# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#

import os
import re
import sys
from optparse import OptionParser

help_text = """
Adapts inter-workspace motions in any wmctrl-compatible window manager as if the
workspaces are positioned on a X_WORKSPACESxY_WORKSPACES flat torus: periodic in
x- and y-coordinate. Indexing is row-major. Defaults to 2x2 workspaces.

Usage:
    Install wmctrl;
    to jump to an adjacent workspace, do
>   wspacetorus.py -d DIRECTION
    with DIRECTION in ['left', 'right', 'top', 'bottom']; to move the current
    window to such workspace, do
>   wspacetorus.py -t DIRECTION
    to use with 3x2 (columns-rows) grid, do
>   wspacetorus.py -t DIRECTION -x 3 -y 2

To see more options, call
>   wpacetorus.py -h

Check https://github.com/OVlasiuk/snippets for updates.
You may also want to look at the plugin
https://cinnamon-spices.linuxmint.com/applets/view/116
"""

if len(sys.argv) < 2:
    print(help_text)
    exit()


def get_current_desktop():
    desktopstring = os.popen('wmctrl -d').read()
    regcurrent = re.compile(r"\d+.*\*")
    regdigits = re.compile(r"\d+")
    strcurrentspan = regcurrent.search(desktopstring).span()
    strcurrent = desktopstring[strcurrentspan[0]:strcurrentspan[1]]
    strdigitsspan = regdigits.match(strcurrent).span()
    strdigits = strcurrent[strdigitsspan[0]:strdigitsspan[1]]
    return int(strdigits)


def set_current_desktop(n):
    s = str(n)
    os.system('wmctrl -s ' + s)


def move_to_desktop(n):
    s = str(n)
    os.system('wmctrl -r :ACTIVE: -t ' + s + ' && wmctrl -s ' + s)


def get_options(parser):
    """ Define command line options."""
    parser.add_option(
        "-d", "--desktop", dest="desktop", default=None,
        help="Jump to an adjacent workspace, given by 'up', 'down', 'left',\
        'right'.")
    parser.add_option(
        "-t",
        "--todesktop",
        dest="todesktop",
        default=None,
        help="Move the current window to an adjacent workspace, given by 'up',\
        'down', 'left', 'right'.")
    parser.add_option(
        "-x",
        "--xworkspaces",
        dest="xworkspaces",
        default=2,
        help="The number of columns in the workspace grid. Default: 2.")
    parser.add_option(
        "-y",
        "--yworkspaces",
        dest="yworkspaces",
        default=2,
        help="The number of rows in the workspace grid. Default: 2.")
    options, args = parser.parse_args()
    return options.desktop, options.todesktop, options.xworkspaces,\
        options.yworkspaces


def numerize(inp, X_WORKSPACES):
    return {
        'left': X_WORKSPACES-1,
        'right': 1,
        'up': -X_WORKSPACES,
        'down': X_WORKSPACES
    }.get(inp, 0)


def gridparse(cdesk, X_WORKSPACES):
    cdeskx = cdesk % X_WORKSPACES
    cdesky = (cdesk - cdeskx) // X_WORKSPACES
    return cdeskx, cdesky


def main(desktop, todesktop, X_WORKSPACES, Y_WORKSPACES):
    if desktop is not None:
        cdesk = get_current_desktop()
        cdeskx, cdesky = gridparse(cdesk, X_WORKSPACES)
        delta = numerize(desktop, X_WORKSPACES)
        deltax, deltay = gridparse(delta, X_WORKSPACES)
        cdeskx = (cdeskx + deltax) % X_WORKSPACES
        cdesky = (cdesky + deltay) % Y_WORKSPACES
        cdesk = cdesky * X_WORKSPACES + cdeskx
        set_current_desktop(cdesk)
    elif todesktop is not None:
        cdesk = get_current_desktop()
        cdeskx, cdesky = gridparse(cdesk, X_WORKSPACES)
        delta = numerize(todesktop, X_WORKSPACES)
        deltax, deltay = gridparse(delta, X_WORKSPACES)
        cdeskx = (cdeskx + deltax) % X_WORKSPACES
        cdesky = (cdesky + deltay) % Y_WORKSPACES
        cdesk = cdesky * X_WORKSPACES + cdeskx
        move_to_desktop(cdesk)

if __name__ == "__main__":
    parser = OptionParser()
    desktop, todesktop, X_WORKSPACES, Y_WORKSPACES = get_options(parser)
    main(desktop, todesktop, int(X_WORKSPACES), int(Y_WORKSPACES))
{% endraw %}
{% endhighlight %}

---
layout: post
title: Wayland Rice - part III
abstract: Is there a better approach to ad hoc terminals in a tiling wm? I think I found one that I quite like.
tags: linux desktop
---

Is there a better approach to ad hoc terminals in a tiling wm? I think I found one that I quite like.

## Huh?

I have a lot of terminals open at any point in time. A number are for long running applications and live in well known positions on well known workspaces; things like newsboat, irssi etc. But I use a lot for ad hoc tasks - ranger, pacman, vim, random bits of shell like searching files and text and so on. It's the ad hoc ones that sort of get in the way of the main purpose of that workspace when they're on it, and never quite seem to be on the workspace that I happen to be on when I want them again. This results in a bunch of switching to different workspaces, retagging views too often or having my nice layouts spoiled. There should be a better UX for this. 

## Drop Down Terminals

To be honest, this issue had sort of bugged me only a little bit, and I really hadn't given it that much thought until I saw a brief conversation in the `#river` channel on IRC where someone was asking about a similar problem they were trying to solve and one of the answers included something about [drop down terminals](https://www.addictivetips.com/ubuntu-linux-tips/best-drop-down-terminal-apps-linux/). It struck me that the same thing would be the answer to my awkward ad hoc terminal UX **if** I could make one of these work in river the way I wanted it to.

The idea and name of these comes originally from the classic video game, Quake, where a command terminal could be accessed in the game by hitting F12 and the terminal would slide down from the top of the screen. But why does it matter now, and how would it potentially solve my problem? If the terminal can be shown and hidden, and it keeps it's state/context across workspaces (i.e. I can activate it on one space, run a pacman upgrade, move to another workspace and be able to check on the progress or interact with the occasional prompts) that would be helpful.

There are two general issues that must be overcome (and a bunch of "specific to my tastes" issues too). In general, these types of terminal emulators are designed to interact with a desktop environment such as Gnome, KDE, LXDE or similar. The terminal cooperates with the desktop and its key binding and window management functions quite tightly in order to work. Here I use no such environment and so my terminal will need to operate in isolation and be configurable via river key mappings and rules. 

## Tilda

I looked at two or three, and Tilda seemed promising as it had a dbus interface meaning it should be flexible enough to work in my stack. After some trial and error, here's what I ended up with.

![tilda]({{ site.url }}/public/images/wayland/tilda.png)

With `Super+T` tilda appears (I made it on the left rather than the top) and you can open multiple tabs. Switching workspaces keeps tilda visible but if it loses focus, it will auto-hide after a couple of seconds. A further `Super+T` brings it back intact. If you're used to closing terminals with `Control+D` as I am, I've configured it to auto-spawn a new terminal and hide if the last tab is closed this way, rather than exit the daemon. So I get the behaviour that feels like common sense to me when I do that. All of the above requires custom config across tilda and river.

### River Init

In my river `init` file, I've added the following;

```bash
# main key combo to toggle visibility of tilda
riverctl map normal Super T spawn "tilda -T"

# sets tilda to float, be visible on all 'workspaces' and appear on the left below the bar
riverctl rule-add -app-id "Tilda*" float
riverctl rule-add -app-id "Tilda*" tags 65535
riverctl rule-add -app-id "Tilda*" position 0 40

# start tilda with dbus so that river can toggle it as we have no Gnome like window manager
tilda --dbus &
```

### Tilda Config

These are the key bits of tilda config to make it work with river. You can change these in the preferences UI or by editing `~/.config/tilda/config_0`. All other settings you can adjust to taste.

```bash
# don't have a global quit option
quit_key="NULL" 

# note that tilda's x_pos and y_pos will be ignored, river sets the window position
x_pos=0
y_pos=0

# hide on startup and focus loss, and spawn a new terminal if the last tab is removed with Ctrl-d
hidden=true
start_fullscreen=false
auto_hide_time=2000
auto_hide_on_focus_lost=true
auto_hide_on_mouse_leave=false
on_last_terminal_exit=2
prompt_on_exit=false
```

..and finally, dracula theme of course :) 

```bash
enable_transparency=true
palette_scheme=0
palette={9766, 9766, 9766, 58339, 22102, 42919, 16962, 59110, 27756, 58596, 62451, 19018, 39835, 27499, 57311, 59110, 18247, 18247, 30069, 56540, 60652, 61423, 42405, 21588, 31354, 31354, 31354, 65535, 31097, 50886, 20560, 64250, 31611, 61937, 63736, 44204, 48573, 37779, 63993, 65535, 21845, 21845, 35723, 59881, 65021, 65535, 47288, 27756}
```

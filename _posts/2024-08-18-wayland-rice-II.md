---
layout: post
title: Wayland Rice - part II
abstract: Thanks to all the brilliant upstream work from wayland/mesa/linux/nvidia devs, it looks like Wayland is fully usable for me now - time to get to work replicating the desktop experience I refined over many months in bspwm, sxhkd, polybar and others.
tags: linux desktop
---

Thanks to all the brilliant upstream work from wayland/mesa/linux/nvidia devs, it looks like Wayland is fully [usable]({{ site.url }}/2024/08/13/wayland-rice-I.html) for me now - time to get to work replicating the desktop experience I refined over many months in bspwm, sxhkd, polybar and others.


## General Theming

I like and use the [dracula](https://draculatheme.com/) dark theme palette, and with so many available configs ready to go for it, you can quickly and easily make a very consistent and integrated looking desktop environment with it, regardless of the mix of software tools you choose. This is the end result; I'll break down all the main apps and configs with some additional sweetners on the way ![desktop]({{ site.url }}/public/images/wayland/desktop.png)
_Note: windows are all set to floating here just to highlight the theming a bit better, normally they would all be tiled other than the rofi application launcher_


## River

Let's start with [riverwm](https://codeberg.org/river/river) - the wayland compositor and tiling window manager I chose. Initially I planned to look at river and hyprland after discovering that sway was opinionated about a couple of things that I had different opinions on. But I never got as far as hyprland because I loved the initial foray with river so much. Seems hyprland got more mind share than river though - I hope Isaac and the rest of the river community have enough momentum to keep it going!

### Launching river

Once logged in on a tty you can just run `river` to launch it. Graphical logins are so overkill. But I'm also too lazy even for that, so I have this in my `fish` config (convert to bash and add to `.profile` or similar)

```bash
if [ -z "$WAYLAND_DISPLAY" ] && [ $(tty) = "/dev/tty1" ]
  exec river
end
```

.. which checks that a Wayland compositor isn't already running and that I'm on `tty1`, if so it starts river up.

### River Init

Similar to bspwm, river is basically configured via an executable invoking a control program (`riverctl`). Shell script is easiest for me, but you can use anything that can delegate to the control binary. The docs are pretty comprehensive and the supplied [example init](https://github.com/riverwm/river/blob/master/example/init) file sets good defaults and is well commented so it's a great place to start. Apart from some changes to key mappings to make it more similar to what I was familiar with in sxhkd, here are the key things I changed from the defaults.

```bash
# ----------------------------------------------------------------------------
# Startup apps and services
# ----------------------------------------------------------------------------
import-gsettings &
dunst &
launch-waybar &
pgrep -x greenclip > /dev/null || /usr/bin/greenclip daemon &
squeezelite -n Office &
sway-rotate-wallpapers.sh &
swayidle -w timeout 1230 "notify-send -u low System 'Locking session in 30 seconds'" \
    timeout 1260 "swaylock-desktop-image.sh" \
    timeout 1800 "wlopm --off '*'" resume "wlopm --on '*'" &
brave --profile-directory=Default &
brave --profile-directory=Work & 
signal-desktop &
```

Many of the above are covered in sections below, but I basically ensure a bunch of things run as soon as river starts.

The [swayidle](https://github.com/swaywm/swayidle) config runs a background process to handle idle time. In my case, it waits about 20 minutes before sending a low urgency notification to the desktop warning of imminent lock (so I can move the mouse or press a key if I've been sat watching a video and want to reset the idle timer). 30s later it will run another custom script to lock the desktop: this is a common idea where it takes a screenshot, fuzzes and blurs it for privacy and sets it as the background for the lock screen. If the machine remains idle for another 10 minutes, the display is turned off by [wlopm](https://sr.ht/~leon_plickat/wlopm/). Swaylock theming is shown further below in the main theming section.


### Tiler

The other key part of my river `init` was replacing the default tiler with something that was a bit closer to `bspwm`. You should play with the default one to understand the basics and see what you like and what you don't. For me, the killer was having to have the same layout applied across all "workspaces" (tags). I need different layouts per tag. [wideriver](https://github.com/alex-courtis/wideriver) is what I'm currently using, but maybe something else will appear that works better for me :) 

River `init` code for the tiler, with border colors from the dracula scheme:

```bash
# ----------------------------------------------------------------------------
# Layout manager, window settings and launcher
# ----------------------------------------------------------------------------
riverctl map normal Super J focus-view next
riverctl map normal Super K focus-view previous
riverctl map normal Super+Shift J swap next
riverctl map normal Super+Shift K swap previous

riverctl map normal Super F toggle-float
riverctl map normal Super Z toggle-fullscreen
riverctl map normal Super N zoom

riverctl map normal Super Left send-layout-cmd wideriver "--layout left"
riverctl map normal Super Right send-layout-cmd wideriver "--layout right"
riverctl map normal Super H send-layout-cmd wideriver "--ratio -0.05"
riverctl map normal Super L send-layout-cmd wideriver "--ratio +0.05"
riverctl default-layout wideriver

wideriver \
    --layout                       left        \
    --layout-alt                   monocle     \
    --stack                        dwindle     \
    --count-master                 1           \
    --ratio-master                 0.50        \
    --count-wide-left              0           \
    --ratio-wide                   0.35        \
    --no-smart-gaps                            \
    --inner-gaps                   3           \
    --outer-gaps                   3           \
    --border-width                 2           \
    --border-width-monocle         2           \
    --border-width-smart-gaps      3           \
    --border-color-focused         "0x6272a4"  \
    --border-color-focused-monocle "0x586e75"  \
    --border-color-unfocused       "0x282a36"  \
    --log-threshold                info        \
   > "$HOME/wideriver.log" 2>&1 &
```

## Waybar 

[Waybar](https://github.com/Alexays/Waybar) is a highly configurable, extensible and themable bar for a number of wayland compositors. It's well documented and easy enough to play with it to get the setup and the modules you want - I'll just add the bits of config here relevant to this rice. To get the fonts and colors, I start the waybar `style.css` with:

```css
@import url("./colors.css");

* {
    /* `otf-font-awesome` is required to be installed for icons */
    font-family: FontAwesome, Roboto, sans-serif;
    font-size: 20px;
}

window#waybar {
    background-color: @background-alpha;
    color: @foreground;
    transition-property: background-color;
    transition-duration: .5s; 
}
```
_relevant distro packages or other installs will be required to get the fonts_

The `colors.css` file in the same directory just defines the dracula palette 

```css
@define-color background-alpha rgba(0, 0, 0, 0.6);
@define-color background #282a36;
@define-color selection #44475a;
@define-color foreground #f8f8f2;
@define-color comment #6272a4;
@define-color cyan #8be9fd;
@define-color green #50fa7b;
@define-color orange #ffb86c;
@define-color pink #ff79c6;
@define-color purple #bd93f9;
@define-color red #ff5555;
@define-color yellow #f1fa8c;
```

Tags (virtual desktops) in river I have on the left of the bar, my music player in the centre and everything else to the right. Layout config is

```jsonc
    "modules-left": ["river/tags"],
    "modules-center": ["custom/squeezebox"],
    "modules-right": ["custom/pacman", "pulseaudio", "disk#root", "disk#home", "cpu", "memory", "temperature", "keyboard-state", "clock", "tray"],
```

`river/tags` are defined in the river `init` file and are per the default from the example config. They are styled as:

```css
button {
    box-shadow: inset 0 -3px transparent;
    border: none;
    border-radius: 0;
    padding: 0 10px;
    color: @comment;
}
button:hover {
    background: inherit;
    color: @foreground;
}
button.focused {
    color: @foreground;
    box-shadow: inset 0 -3px @pink;
}
button.urgent {
    background-color: @red;
}
```

![waybar-tags]({{ site.url }}/public/images/wayland/waybar-tags.png)

There's not much to the styling of the modules on the right, other than to set the appropriate foreground `color` property in the `style.css` for the relevant module.

![waybar-right]({{ site.url }}/public/images/wayland/waybar-right.png)


### Custom Waybar Modules

Code for my trivial custom [waybar-modules](https://github.com/davison/waybar-modules) is available on github.

I use one that just periodically checks which packages need upgrading and outputs the count of them on the bar with a package icon. When you hover over the tooltip shows the actual packages. This works with `pacman` for Arch but could easily be adapted for any other package manager from another distro

![pacman-module]({{ site.url }}/public/images/wayland/pacman-module.png)

The `~/.config/waybar/config` snippet to define it is:

```jsonc
"custom/pacman": {
    "format": "{}",
    "return-type": "json",
    "escape": true,
    "interval": 3600,
    "exec": "$HOME/projects/waybar-modules/pacman/pacman-upgrades",
    "on-click": "alacritty --hold -e 'yay -Syyu'"
}
```

and a bit of custom `style.css`
```css
#custom-pacman {
    color: @red;
}
```

The other module is for my [multi-room audio]({{ site.url }}/2019/03/09/multi-room-audio-I.html) system, based on squeezebox (read the posts on that to see why I chose it!)

![squeezebox-paused]({{ site.url }}/public/images/wayland/squeezebox-paused.png)
![squeezebox-playing]({{ site.url }}/public/images/wayland/squeezebox-playing.png)

Config for the waybar module:

```jsonc
"custom/squeezebox": {
    "interval": 5,
    "format": "🎜 {} {percentage}%",
    "return-type": "json",
    // IP address is the address of your squeezebox server, "Office" here is the name of the player to control
    "exec": "$HOME/waybar-modules/.venv/bin/python $HOME/waybar-modules/squeezebox/squeezebox 10.1.2.3 Office",
    "on-click": "$HOME/waybar-modules/.venv/bin/python $HOME/waybar-modules/squeezebox/squeezebox 10.1.2.3 Office toggle"
}
```

.. the styling simply uses a different foreground colour depending on whether the player is paused/stopped or playing.

```css
#custom-squeezebox.pause, #custom-squeezebox.stop {
    color: @comment;
}
#custom-squeezebox.play {
    color: @green;
}
```

## Additional Theming

I'm using [dunst](https://dunst-project.org/) for desktop notification management, and [rofi](https://github.com/davatorium/rofi) as an application launcher. Both are highly customisable and themable.

### Dunst

Relevant parts of the `dunstrc` config..

```ini

[global]
    monitor = 0
    follow = mouse

    width = 800
    height = 300
    origin = bottom-center

    gap_size = 0
    separator_color = frame
    sort = yes

    font = "Roboto 14"
    line_height = 0

    markup = full
    format = "<b>%s</b>\n%a\n\n%b"
    alignment = left
    vertical_alignment = center
    show_age_threshold = 60

    show_indicators = yes

    enable_recursive_icon_lookup = true
    icon_theme = Tela-purple
    icon_position = left
    min_icon_size = 32
    max_icon_size = 128
    icon_path = /usr/share/icons/Tela-purple/scalable/status/:/usr/share/icons/Tela-purple/scalable/devices/:/usr/share/icons/Tela-purple/scalable/places:/usr/share/icons/Tela-purple/scalable/mimetypes

[urgency_low]
    background = "#282a36"
    foreground = "#f8f8f2"
    frame_color = "#6272a4"
    timeout = 5

[urgency_normal]
    background = "#282a36"
    foreground = "#f8f8f2"
    frame_color = "#50fa7b"
    timeout = 10

[urgency_critical]
    background = "#800000"
    foreground = "#ffffff"
    frame_color = "#ff5555"
    timeout = 0
```

### Rofi

Rofi is well documented, here is my `config.rasi` file..

```css
/*
 * ░█▀▄░█▀█░█▀▀░▀█▀░░░▀█▀░█░█░█▀▀░█▄█░█▀▀
 * ░█▀▄░█░█░█▀▀░░█░░░░░█░░█▀█░█▀▀░█░█░█▀▀
 * ░▀░▀░▀▀▀░▀░░░▀▀▀░░░░▀░░▀░▀░▀▀▀░▀░▀░▀▀▀
 */

configuration {
    ssh-command: "alacritty -e \"{ssh-client} {host} [-p {port}]\"";
}

@theme "arthur"

* {
    drac-bgd: #282a36;
    drac-bgd-t: #282a36cc;
    drac-cur: #44475a;
    drac-fgd: #f8f8f2;
    drac-cmt: #6272a4;
    drac-cya: #8be9fd;
    drac-grn: #50fa7b;
    drac-ora: #ffb86c;
    drac-pnk: #ff79c6;
    drac-pur: #bd93f9;
    drac-red: #ff5555;
    drac-yel: #f1fa8c;

    font: "Roboto 18";
    foreground: @drac-fgd;
}
window {
    width: 800px;
}
inputbar {
    color: @drac-cya;
    background-color: @drac-bgd-t;
    children: [ prompt,textbox-prompt-colon,entry,case-indicator ];
}
textbox-prompt-colon {
    expand:     false;
    str:        ":";
    margin:     0px 0.3em 0em 0em ;
    text-color: @drac-grn;
}
prompt,case-indicator {
    text-color: @drac-grn;
}
listview {
    background-color: @drac-bgd-t;
}
entry, prompt {
    font: "Roboto 20";
}
element {
    orientation: horizontal;
    children: [ element-icon, element-text ];
    spacing: 5px;
}
element {
    orientation: horizontal;
    children: [ element-icon, element-text ];
    spacing: 5px;
}
element selected.normal {
    background-color: @drac-pnk;
}
element normal active {
    foreground: @drac-yel;
}
element normal urgent {
    foreground: @drac-red;
}
element alternate active {
    foreground: @drac-ora;
    foreground: @drac-bgd;
}
element alternate urgent {
    foreground: @drac-red;
}
element selected active {
    background-color: @drac-ora;
    foreground: @drac-bgd;
}
element selected urgent {
    background-color: @drac-red;
    foreground: @drac-bgd;
}
```

and the relevant keybindings from `river/init`..

```bash
# Rofi
riverctl map normal Super Space spawn "rofi -modi run,drun,window,ssh -show drun"
riverctl map normal Super X spawn "rofi -modi \"clipboard:greenclip print\" -show clipboard"
```

### GTK

I'm using the [dracula gtk](https://draculatheme.com/gtk) theme, but not the icons from the same place - I'm using [Tela](https://github.com/vinceliuice/Tela-icon-theme) purple dark icons instead. You can use something like [lxappearance](https://github.com/lxde/lxappearance) to help configure the various GTK settings files, or just update them manually.

From `~/.gtkrc-2.0`..

```ini
gtk-theme-name="dracula"
gtk-icon-theme-name="Tela-purple-dark"
```

and from `~/.config/gtk-3.0/settings.ini`

```ini
[Settings]
gtk-application-prefer-dark-theme=1
gtk-theme-name=dracula
gtk-icon-theme-name=Tela-purple-dark
```

The `import-gsettings` script, called at the top of the river `init` script, is a hacky workaround for some gnome/gtk software theming issues, I can't recall where I found this now but credit goes to someone else for it...

```bash
config="${XDG_CONFIG_HOME:-$HOME/.config}/gtk-3.0/settings.ini"
if [ ! -f "$config" ]; then exit 1; fi

gnome_schema="org.gnome.desktop.interface"
gtk_theme="$(grep 'gtk-theme-name' "$config" | sed 's/.*\s*=\s*//')"
icon_theme="$(grep 'gtk-icon-theme-name' "$config" | sed 's/.*\s*=\s*//')"
cursor_theme="$(grep 'gtk-cursor-theme-name' "$config" | sed 's/.*\s*=\s*//')"
font_name="$(grep 'gtk-font-name' "$config" | sed 's/.*\s*=\s*//')"
gsettings set "$gnome_schema" gtk-theme "$gtk_theme"
gsettings set "$gnome_schema" icon-theme "$icon_theme"
gsettings set "$gnome_schema" cursor-theme "$cursor_theme"
gsettings set "$gnome_schema" font-name "$font_name"
```


### Wallpapers

I use another script, launched from the river `init` file, to rotate the wallpaper on the desktop. The wallpaper images are all in a single directory and this script picks a random one every 15 minutes from those available then uses [swaybg](https://github.com/swaywm/swaybg) to display it.

My wallpapers are either predominantly purple, dracula/vampire themed, or both :) 

![wallpapers]({{ site.url }}/public/images/wayland/wallpapers.png)

```bash
#!/usr/bin/env bash
#
# ░█░█░█▀█░█░░░█░░░█▀█░█▀█░█▀█░█▀▀░█▀▄░█▀▀
# ░█▄█░█▀█░█░░░█░░░█▀▀░█▀█░█▀▀░█▀▀░█▀▄░▀▀█
# ░▀░▀░▀░▀░▀▀▀░▀▀▀░▀░░░▀░▀░▀░░░▀▀▀░▀░▀░▀▀▀
#

src_dir=$HOME/Pictures/wallpapers/ 
interval=900 # seconds

killall swaybg
swaybg -i $(find $src_dir -maxdepth 1 -type f | shuf -n1) -m fill &
old_pid=$!

while true; do
    sleep $interval
    swaybg -i $(find $src_dir -maxdepth 1 -type f | shuf -n1) -m fill &
    next_pid=$!
    sleep 5
    kill $old_pid
    old_pid=$next_pid
done
```


### Swaylock

Here's the `~/.local/bin/swaylock-desktop-image.sh` code touched on earlier.. some of the effects here you can get from an additional swaylock package (`swaylock-effects`) that you might want to checkout instead.

```bash
#!/bin/bash
#
# ░█▀▀░█░█░█▀█░█░█░█░░░█▀█░█▀▀░█░█
# ░▀▀█░█▄█░█▀█░░█░░█░░░█░█░█░░░█▀▄
# ░▀▀▀░▀░▀░▀░▀░░▀░░▀▀▀░▀▀▀░▀▀▀░▀░▀
#
# create a lock screen image from the current desktop background for swaylock
# to use.
#
# ----------------------------------------------------------------------------

tgt_dir=$HOME/Pictures
tgt_file=${tgt_dir}/screenlockimage.png

mkdir -p ${tgt_dir}

if [ $(command -v grim mogrify | wc -l) -eq 2 ]; then 
    rm -f ${tgt_file}
    grim ${tgt_file}
    mogrify -scale 5% -scale 2000% ${tgt_file}
    mogrify -blur 0 ${tgt_file}
    swaylock -i ${tgt_file}
else
    swaylock
fi
```

..and my swaylock config (`~/.config/swaylock/config`) with dracula theme colors

```ini
#
# ░█▀▀░█░█░█▀█░█░█░█░░░█▀█░█▀▀░█░█
# ░▀▀█░█▄█░█▀█░░█░░█░░░█░█░█░░░█▀▄
# ░▀▀▀░▀░▀░▀░▀░░▀░░▀▀▀░▀▀▀░▀▀▀░▀░▀
#                                

ignore-empty-password
show-failed-attempts
daemonize
scaling=fill
font=Roboto
indicator-radius=250
indicator-thickness=25
line-uses-ring

bs-hl-color=ff79c6
key-hl-color=50fa7b
separator-color=f8f8f2
color=282a36
inside-color=282a36cc
ring-color=bd93f9
text-color=f8f8f2

inside-clear-color=282a36cc
ring-clear-color=f1fa8c
text-clear-color=f8f8f2

inside-wrong-color=282a36cc
ring-wrong-color=ff5555
text-wrong-color=f8f8f2

inside-ver-color=282a36cc
ring-ver-color=50fa7b
text-ver-color=f8f8f2

```

I also have the script bound to a key combo in river `init` so that I can invoke it manually if I step away from the machine.

```bash
riverctl map normal Control+Alt Backspace spawn swaylock-desktop-image.sh
```

and here are the lock screens with 4 states of 

1. unlock in progress
![unlocking]({{ site.url }}/public/images/wayland/swaylock2.png)

1. verifying
![verifying]({{ site.url }}/public/images/wayland/swaylock1.png)

1. incorrect password
![wrong]({{ site.url }}/public/images/wayland/swaylock3.png)

1. input cleared
![clear]({{ site.url }}/public/images/wayland/swaylock4.png)


## Wrap Up

I've had a good time configuring the desktop so far - probably some more to come. But for now, it's at least as good looking as my previous Xorg based setup was, better in places, and I'm happy with everything from usability to look & feel to performance. Hope you have as much fun ricing your own :) 


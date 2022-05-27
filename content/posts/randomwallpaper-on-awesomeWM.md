---
title: "Random wallpaper with AwesomeWM !"
date: 2020-01-10T11:10:33Z
draft: False
type: "post"
tags: [awesomewm, linux, wallpaper, random]
---

I've been looking for a way to break my routine a bit when I'm working on my laptop. I figured that changing the wallpaper randomly and automatically was a good way to break the monotony.

I use awesomeWM (version 4)
```
[f00b@void ~]$ awesome --version
awesome v4.3 (Too long)
 • Compiled against Lua 5.3.5 (running with Lua 5.3)
 • D-Bus support: ✔
 • execinfo support: ✘
 • xcb-randr version: 1.6
 • LGI version: 0.9.2
```

So I needed three things:

- A folder full of images
- A little script that will choose one at random
- A call to this script from awesome init

For the images, I use [the excellent repo](https://github.com/LukeSmithxyz/wallpapers) by **Luck Smith**.

As far as the script is concerned, nothing too hard:
```
#!/bin/bash
# author : Kaderovski
# descr : Make your wallpaper change on each start !
#
# I'm using Luck Smith wallpaper git repo for all images
# link : https://github.com/LukeSmithxyz/wallpapers

# current awesome theme
THEME="powerarrow-dark"
# Awesome conf path
AWPATH="$HOME/.config/awesome/themes/$THEME"
# image should have absolute path to image folder
IMAGE=$(find $HOME/Pictures/wallpapers/ -type f -name "*.png" -o -name "*.jpeg" -o -name "*.jpg"| shuf -n 1 | sed 's/\ /\\ /g')

cp -f $IMAGE $AWPATH/wall.png

# don't forget to add those lines at the end of your rc.lua (replace with your correct path and script name)
#
# -- Startup programs
# awful.util.spawn_with_shell("~/bin/wallpaper.sh")
```

As indicated in comment, just add these two lines (or just one without the comment) to call the script via awesomeWM init.

Put this at the end of `~/.config/awesome/rc.lua`.
```
-- Startup programs
awful.util.spawn_with_shell("~/bin/wallpaper.sh")
```

On each awesomewm restart, you will have a new pretty (or not) wallpaper.

[Script is on gist !](https://gist.github.com/Kaderovski/785796c5e4e37a4d759f0f8c89c41f0e)

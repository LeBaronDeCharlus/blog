---
title: "Shloader - A Modern Shell Loader"
date: 2022-06-25T18:03:17+02:00
image: /images/shloader.png 
tags: [loader, shell, bash]
type: "post"
weight: 25
showTableOfContents: true
draft: false
---

![Shloader](/images/shloader.png)

1. [Why](#why)
2. [Features](#features)
3. [Preview](#preview)
4. [Templating](#templating)
5. [Usage](#usage)
6. [Parsing Arguments](#parsing-arguments)
7. [Shell Configurations](#shell-configurations)
8. [Trap Error and Exit](#trap-error-and-exit)
9. [Display Loader](#display-loader)
10. [Call Loader](#call-loader)
11. [Scrip Library Integration](#script-library-integration)
12. [Conclusion](#conclusion)

# Why

I've been recently doing some old shell scripts to quickly automatize and distribute actions that were made manually, I'll write a post on it later. For now I'd like to share with you how I've been coding a 100% shell loader library to use it in my scripts.

I've been looking on the net for existing library, I've found some interesting Github projects but it doesn't appear to be "library" ready. Furthermore, it was mostly old basic ASCII templates and I wanted to give a try to modern loaders and spinners on shell.

So I ended create [`shloader`](https://github.com/kaderovski/shloader).

# Features


shloader has nice features such as :

😍 emoji support

💪 loader support

😎 dynamic message on load step

ℹ️  message on step ending

🎨 multiple loading templates

👌 light and easy to use on existing scripts

# Preview

![Shloader](/images/shloader.gif)

# Templating

First of all, we need to create templates, one for each loaders that will be displayed. I've decided to use shell `array` so we can work on iteration later on customs functions.

So I ended with something like :
```shell
# EMOJIS
emoji_hour=( 0.08 '🕛' '🕐' '🕑' '🕒' '🕓' '🕔' '🕕' '🕖' '🕗' '🕘' '🕙' '🕚')
emoji_face=( 0.08 '😐' '😀' '😍' '🙄' '😒' '😨' '😡')
emoji_earth=( 0.1 🌍 🌎 🌏 )
emoji_moon=( 0.08 🌑 🌒 🌓 🌔 🌕 🌖 🌗 🌘 )
emoji_orange_pulse=( 0.1 🔸 🔶 🟠 🟠 🔶 )
emoji_blue_pulse=( 0.1 🔹 🔷 🔵 🔵 🔷 )
emoji_blink=( 0.06 😐 😐 😐 😐 😐 😐 😐 😐 😐 😑 )
emoji_camera=( 0.05 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📷 📸 📷 📸 )
emoji_sick=( 0.2 🤢 🤢 🤮 )
emoji_monkey=( 0.2 🙉 🙈 🙊 🙈 )
emoji_bomb=( 0.2 '💣   ' ' 💣  ' '  💣 ' '   💣' '   💣' '   💣' '   💣' '   💣' '   💥' '    ' '    ' )
# ASCII
ball=( 0.2 '(●)' '(⚬)')
arrow=( 0.06 '↑' '↗' '→' '↘' '↓' '↙' '←' '↖')
cym=( 0.1 '⊏' '⊓' '⊐' '⊔')
x_plus=( 0.08 '×' '+')
line=( 0.08 '☰' '☱' '☳' '☷' '☶' '☴')
ball_wave=( 0.1 '𓃉𓃉𓃉' '𓃉𓃉∘' '𓃉∘°' '∘°∘' '°∘𓃉' '∘𓃉𓃉')
old=( 0.07 '—' "\\" '|' '/' )
dots=( 0.04 '⣾' '⣽' '⣻' '⢿' '⡿' '⣟' '⣯' '⣷' )
dots2=( 0.04 '⠋' '⠙' '⠹' '⠸' '⠼' '⠴' '⠦' '⠧' '⠇' '⠏' )
dots3=( 0.04 '⠋' '⠙' '⠚' '⠞' '⠖' '⠦' '⠴' '⠲' '⠳' '⠓' )
dots4=( 0.04 '⠄' '⠆' '⠇' '⠋' '⠙' '⠸' '⠰' '⠠' '⠰' '⠸' '⠙' '⠋' '⠇' '⠆' )
dots5=( 0.04 '⠋' '⠙' '⠚' '⠒' '⠂' '⠂' '⠒' '⠲' '⠴' '⠦' '⠖' '⠒' '⠐' '⠐' '⠒' '⠓' '⠋' )
dots6=( 0.04 '⠁' '⠉' '⠙' '⠚' '⠒' '⠂' '⠂' '⠒' '⠲' '⠴' '⠤' '⠄' '⠄' '⠤' '⠴' '⠲' '⠒' '⠂' '⠂' '⠒' '⠚' '⠙' '⠉' '⠁' )
dots7=( 0.04 '⠈' '⠉' '⠋' '⠓' '⠒' '⠐' '⠐' '⠒' '⠖' '⠦' '⠤' '⠠' '⠠' '⠤' '⠦' '⠖' '⠒' '⠐' '⠐' '⠒' '⠓' '⠋' '⠉' '⠈' )
dots8=( 0.04 '⠁' '⠁' '⠉' '⠙' '⠚' '⠒' '⠂' '⠂' '⠒' '⠲' '⠴' '⠤' '⠄' '⠄' '⠤' '⠠' '⠠' '⠤' '⠦' '⠖' '⠒' '⠐' '⠐' '⠒' '⠓' '⠋' '⠉' '⠈' '⠈' )
dots9=( 0.04  '⢹' '⢺' '⢼' '⣸' '⣇' '⡧' '⡗' '⡏' )
dots10=( 0.04  '⢄' '⢂' '⢁' '⡁' '⡈' '⡐' '⡠' )
dots11=( 0.04 '⠁' '⠂' '⠄' '⡀' '⢀' '⠠' '⠐' '⠈' )
```

Array composition is divided in two parts, the first one is a `time` interval in seconds and the second one is the loader frame transition.

# Usage

I wanted library to be used as commands can be, by passing argument through options.

First, let's work on default `usage` function to display to user.

```shell
usage() {
  cat <<EOF
Available options:
-h, --help            <OPTIONAL>    Print this help and exit
-l, --loader          <OPTIONAL>    Chose loader to display
-m, --message         <OPTIONAL>    Text to display while loading
-e, --ending          <OPTIONAL>    Text to display when finishing
EOF
  exit 0
}
```

I wanted to define argument options as bellow :

#### Display help

```shell
# e.g
shloader -h
```

| Parameter | Type     | Description                |
| :-------- | :------- | :------------------------- |
| `-h --help` | `none` | Optional. Show help usage |

#### Chose loader

```shell
# e.g
shloader -l arrow
```

| Parameter | Type     | Description                |
| :-------- | :------- | :------------------------- |
| `-l --loader` | `string` | Optional. Chose loader template|

#### Display info on loading

```shell
# e.g
shloader -m "my loading message" 
```

| Parameter | Type     | Description                |
| :-------- | :------- | :------------------------- |
| `-m --message` | `string` | Optional. Show a text message while displaying loader|

#### Display info on ending

```shell
# e.g
shloader -e "\u2728 all done"
```

| Parameter | Type     | Description                |
| :-------- | :------- | :------------------------- |
| `-e --end` | `string` | Optional. Show an end text message when loader ends|

# Parsing Arguments

In order to work on user input, it is necessary to read option content so we can modular library execution.

```shell
shloader() {
  loader=''
  message=''
  ending=''

  while :; do
    case "${1-}" in
    -h | --help) usage;;
    -l | --loader)
      loader="${2-}"
      shift
      ;;
    -m | --message)
      message="${2-}"
      shift
      ;;
    -e | --ending)
      ending="${2-}"
      shift
      ;;
    -?*) die "Unknown option: $1" ;;
    *) break ;;
    esac
    shift
  done

[…]
}
```

You might notice I've placed it directly in the main `function` so it will be the first element to be run.

If user don't specify loader, we should display one as default, let's say `dots` one.

```shell
# shloader func
[…]

  if [[ -z "${loader}" ]] ; then
    loader=dots[@] 
  else
    loader=$loader[@]
  fi

[…]
```

One last thing here, I use a little custom `die` function to exit properly if option is unknown.

```shell
die() {
  local code=${2-1}
  exit "$code"
}
```

# Shell Configurations

We will make some quick configuration on the top header script under `shebang`.

```shell
#!/bin/bash
# https://github.com/Kaderovski/shloader
# me@kaderovski.com
set -Eeuo pipefail
trap end_shloader SIGINT SIGTERM ERR EXIT RETURN
tput civis
```

First, `set -Eeuo pipefail` so we can exit execution if one of the commands in the pipe fails.

Then, `trap end_shloader SIGINT SIGTERM ERR EXIT RETURN` so we can call a custom function to clean our execution on exiting, stopping, errors…

Finaly, `tput civis` is used to hide cursor.

# Trap Error and Exit

Trap will call a custom function, but how does it work ?

```shell
end_shloader() {
  kill "${shloader_pid}" &>/dev/null
  tput cnorm
  if [[ "${ending}" ]]; then
    printf "\r${ending}"; echo
  fi
}
```

Right here I ensure first to kill loader's `PID` and restore cursor thanks to `tput cnorm`.

# Display Loader

Now we can interact with shell output to generate loading animations.

Let's create a new function `play_shloader()` :

```shell
play_shloader() {
  while true ; do
    for frame in "${loader[@]}" ; do
      printf "\r%s" "${frame} ${message}"
      sleep "${speed}"
    done
  done
}
```

Here we assume we have `loader` array we can display during time duration defined in `speed` variable (we will see it after).

# Call Loader

What about now ? We have all bricks to make great loaders, let's put them in work together !

```shell
# shloader function
[…]

  loader=( ${!loader} )
  speed="${loader[0]}"
  unset "loader[0]"

  tput civis
  play_shloader &
  shloader_pid="${!}"

[…]
```

What is done here ?

We first print `loader` array content we save in `loader` variable. We know first element in array is time duration, so we save it in `speed` variable.

Now we have split array, we can remove time duration as it is now under `speed` variable.
We then hide cursor, call our `play_shloader` function and save `PID`.

You can find the [Full Library Code](https://github.com/Kaderovski/shloader/blob/main/lib/shloader.sh)

# Script Library Integration

Nothing hard here !

```shell
source ./lib/shloader.sh

shloader -l emoji_hour -m "Testing" -e "✨ All good !" 

  sleep 2   # remove it in your code
  # … your logic goes here

end_shloader
```

With more details :

```shell
!/bin/bash

# if you want to try just add this block code in your code 
source ./lib/shloader.sh

# you can chose (see more in lib/loader.sh):
# ball, arrow, cym, x_plus, line, ball_wave, npm and old.
# you can specify message to display during loading
# and message to display after your code finished

# eg with npm style
# notice end message -e use unicode emoji to display
# this is for better terminal support
# \u2728 == ✨ but you can use emoji if your settings support it !
shloader -l emoji_face -m "Testing" -e "\u2728 All good !" 
  sleep 2   # remove it in your code

  # … your logic goes here
  # if you want to hide some output from loader
  # don't forget to redirect your STD*
  #
  # eg :
  # STDOUT
  # my_cmd 1> /dev/null
  # STDERR
  # my_cmd 2> /dev/null
  # BOTH
  # my_cmd &> /dev/null

# stop loader
end_shloader
```

# Conclusion

As all things useless… it may become mandatory 😂.

---
title: "B, an ugly Bitwarden CLI"
date: 2023-08-09T13:10:42+02:00
draft: false
tags: [bitwarden, cli, shell]
type: "post"
weight: 25
image: /images/b.png 
---
<br>
<p align="center">
    <img src="/images/b.png" width="30%">
</p>

Yes, yes, yes, I know… I don't like graphical interfaces so sometimes I like to make some TUI tools, and sometimes they can be ugly too…

I was tired of wasting time clicking each time on the web extension or on the client, of having to systematically enter my login details, of not having a quick access directly accessible in shortcuts, etc.

So, living in a terminal 90% of the time, I thought I'd do a quick script to put me out of my misery.

I know, judge me, please yourself :

``` shell
# b
# […]
bw list items --search "${1}" --session "${BW_SESSION}" | jq -j '.[] | [.name, .login.password]' | tr -d '\n["'| tr ']' '\n'| tr ',' ':' | sed 's/^\ \ //g' | "${FZF}" --layout=reverse | tail -1 | tr -d '"' | cut -d ':' -f 2 | sed 's/^\ \ //g'| cat | xclip -se c > /dev/null 2> /dev/null
# […]
```

I needed Bitwarden CLI tool `bw`, a way to copy in clipboard password `xclip`, some json parsing with `jq` and `fzf` (and `fzf-tmux`).

So dependencies are :
- `bw` (Bitwarden cli)
- `xclip`
- `jq`
- `fzf`
- `fzf-tmux`

To install this quick and dirty script, you need to create a Bitwarden token, [check documentation here](https://bitwarden.com/help/personal-api-key/).
Then set variables in b script.

```shell
# Fill these vars
export BW_CLIENTID=''
export BW_CLIENTSECRET=''
export BW_PASSWORD=''
```

Finally move `b` file to `/usr/local/bin`.
```shell
sudo mv b /usr/local/bin 
```

You can then :
- Search for a password
```shell
# ex, search github password
b github
```

- Sync vault (download new password from Bitwarden/Vaultwarden server):
```shell
b --sync
```

And probably a lot more to come as I could be a lazy man.

<a href="https://github.com/lebarondecharlus/b" target="_blank">Sources are here.</a>

---
title: "Manage your Zellij sessions"
date: 2022-07-12T12:22:42+02:00
draft: false
tags: [zellij, sessions, shell]
type: "post"
weight: 25
image: /images/zellij.png 
---

<img src="/images/zellij.png" width="50%">

Quick post on great `Zellij` tool I've been using for some weeks now in `Tmux` replacement.

I've been missing to manage my `sessions` on multiplexer init while starting a new `shell` or `terminal` so I've ended creating a quick feature to manage it on startup.

### Dependencies
You need <a href="https://github.com/lotabout/skim" target="_blank">sk</a> binary installed and in your `$PATH` and of course <a href="https://github.com/zellij-org/zellij/" target="_blank">zellij</a>.

### Demo
![Zellij Sessions](/images/sessions.gif)

### Installation
Add this block at the end of your `$SHELLrc` file <i>(tested with BASH and ZSH)</i> :

```shell
ZJ_SESSIONS=$(zellij list-sessions)
NO_SESSIONS=$(echo "${ZJ_SESSIONS}" | wc -l)

if [ "{$ZELLIJ}" ] && [ -z "${ZELLIJ_SESSION_NAME}" ]; then
  echo -ne "Active Zellij sessions :\n"
  for i in $(echo "${ZJ_SESSIONS}"); do echo -ne "*${i}\n"; done
  echo -ne '\n'
  read REPLY\?"New zellij session [y/n] ? "
  if [ "${REPLY}" = "y" ]; then
    read SESS\?"Session name : "
    zellij --layout compact attach -c "${SESS}"
  else
    if [ "${NO_SESSIONS}" -ge 1 ]; then
      zellij --layout compact attach \
      "$(echo "${ZJ_SESSIONS}" | sk)"
    else
    fi
  fi
fi
```

### Contributing
Feel free to fork and contribute.

<a href="https://github.com/Kaderovski/zellij-sessions" target="_blank">Sources are here.</a>

---
title: "Pulldozer - custom shell modular continous delivery"
date: 2022-06-22T21:53:45+02:00
draft: true
image: /images/pulldozer.png
tags: [git, bash, continous delivery, cd]
type: "post"
weight: 25
showTableOfContents: true
---

![Pulldozer](/images/pulldozer.png)

1. [Why](#why)
2. [Pulldozer](#pulldozer)
3. [Conception](#conception)
4. [Shell loader](#shell-loader)

# Why

In some way, `CD`, continous delivery, as defined today looks too much, in my opinion, like new JS framework, fashion effect. (I'm partially joking, please don't kill me… now). New configurations, "new way", new convention, new format…

It's exhausting to update each time workflow, even if it's for better, what about old classic way that **just works**. What about **full compliant** manifest ?

My point is not to say `CD` new brands technologies are bad, in fact, they help a lot futur programmer to ensure production workflow are not doing blindly. But I'm affraid from time to time, we are forgetting basics, what some call "legacy", "old school".

I don't want to write an elitist post that will blame distributed CD users for using them, I just want to bring back here some `git` and `shell` fundamentals.

# Pulldozer

You might ask… and you can.

# Conception

Where to start and how this service should work ?

I wanted to allow user to chose between single manual run, through `CLI` or `ssh` or `systemd` service. That's not that much, excepting log format that should be compliant with `journalctl`.

Also, this service should be modular, meaning it must :
- be hotly changed by parameter
- be agnostic of whatever rules/payload is set
- run on all UNIX platform
- be easily configured through configuration file

In fact when we talk about `CD`, we want to be sure it won't break production, that's why I wanted `dry-run` option to be set before pulling.

By agnostic, I wanted this script to be working for python, golang, perl, ruby, php… whatever technologies, framework, language used.

Last but not least, I wanted shell script to be sexy, thanks to `loader` and `emojis` support.

# Shell loader

I wanted to recreate npm spinner but in `shell`. Something like this :

![Pulldozer](/images/spinner.svg)

I found some usefull inspiration by looking on [bash_loading_animations](https://github.com/Silejonu/bash_loading_animations) repository, and started by defining animations as list.

```shell
cli_loader=( ⣾ ⣽ ⣻ ⢿ ⡿ ⣟ ⣯ ⣷ )
```

With all these elements, I should be able to create some loading, but speed animation was missing so I've put default speed rotation value at the beginning.


```shell
cli_loader=( 0.05 ⣾ ⣽ ⣻ ⢿ ⡿ ⣟ ⣯ ⣷ )
```



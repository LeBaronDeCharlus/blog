---
title: "Update Proxmox templates"
date: 2017-07-30T14:53:16Z
tags: [proxmox, templates, linux]
type: "post"
draft: false
---

To automatically update the list of templates available via prox in your local space just use the following command:

    pveam update

I found it good to put it in cron once a week so as not to have too much space for minor versions.

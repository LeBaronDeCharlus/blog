---
title: "Additional Volume Public Cloud Ovh"
date: 2017-07-31T11:38:18Z
tags: [ovh, linux, disk]
type: "post"
draft: false
---

123 To add an additional disk/volume on your OVH public cloud, you need to follow some steps.

First, identify your new disk :

    fdisk -l

You can have different output depending of your system (`sd{x}`, `vd{x}`).

Then create a new partition :
```
# parted /dev/{{disk}}
mktable gpt
mkpart primary ext4 512 100%
quit
```

Format it :

    mkfs -t ext4 -L rootfs /dev/{{disk}}1

Mount :

    mount /dev/{{disk}}1 /mnt

Let’s make it peristent, we need **UUID**.
To get block ID.

```
# blkid
/dev/sdb1: LABEL="rootfs" UUID="6b75bbb4-b311-4b9d-a8fd-6e6ff23c401f" TYPE="ext4" PARTLABEL="primary" 
PARTUUID="e20dc227-9d10-41c4-a714-2fb53d190c11"
/dev/sda1: UUID="9abb590f-8a5e-496f-ad2a-2c877415bdc5" TYPE="ext4" PARTUUID="71036eb1-01"
```

Add it on `/etc/fstab` file with mount information.

```
# vim /etc/fstab
[...]
UUID=6b75bbb4-b311-4b9d-a8fd-6e6ff23c401f /mnt ext4 errors=remount-ro,discard 0 1
```

Confirm good configuration with one reboot.


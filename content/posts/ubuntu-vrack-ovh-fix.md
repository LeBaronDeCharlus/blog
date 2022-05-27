---
title: "Ubuntu Vrack Ovh Fix"
date: 2018-03-04T14:41:25Z
draft: false
type: "post"
tags: [bug, ovh, vrack, linux]
---

You may have noticed it, but when you populate a PCI OVH instance under Ubuntu by activating Vrack, your Vm does not have its private IP at boot time.
So, yes, I don't like Ubuntu, but sometimes you don't have a choice.

Anyway, all this to say that we don't have our private IP and it's too sad. (RT)

The fix trick is stupid.
Very stupid.

Add:
```
allow-hotplug ens4
iface ens4 inet dhcp
```
In `/etc/network/interface` file.

Then :
```
systemctl restart network
```
and
```
ifup ens4
```

That’s it

I’ve sent an email on OVH's ML [cloud], because I still found it strange that this bug still exists.

It was on 21 November 2017.

```
Hello la team,

Juste une petite remarque sur l'installation d'un Ubuntu PCI avec Vrack.
En fait je suis obligé d'ajouter :

allow-hotplug ens4
iface ens4 inet dhcp

Dans /etc/network/interface puis

systemctl restart network
et
ifup ens4

Il me semble que sur les autres distrib' les confs network s'ajoutent automatiquement à l'install non ?


Je passe peut-être à côté d'un truc... en même temps je ne connais pas bien les systèmes en .deb...

Bises à tout le monde

F00b4rch
```

Answer : 
```
21/11/2017
	
À cloud
Hello,
Exact, sur debian 9 (voir 8), les interfaces sont montée automatiquement...
Je crois qu'il a un bug d'ouvert cote ubuntu pour ça, si je le retrouve je te l'envoi.
```

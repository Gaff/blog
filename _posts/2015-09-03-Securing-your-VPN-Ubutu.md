---
layout: post
title:  "Securing your VPN on Ubuntu - it's uncomplicated"
date:   2015-09-03 13:44:47
categories: VPN ubuntu ufw
---

## So I've installed my VPN and it's working just fine - what's the problem?

The problem is by default your computer now has two routes to the internet. The regular network and the VPN. While it prefers the VPN there's nothing to prevent it using the regulr route should the VPN cut out for any reason. You won't even know it's happening as most OSs like to hide these low level details from you.

## What can I do to prevent this?

I've seen people use an executioner script that kills your browser if the tunnel cuts out. However a better solution would be a firewall. Ubuntu has a very simple one called [ufw](https://help.ubuntu.com/community/UFW) - Uncomplicated Firewall. And it's really not very complicated. Here's what you need to do:

    sudo ufw enable
    sudo ufw reject outgoing
    sudo ufw allow out 53/udp
    sudo ufw allow out 1194/udp
    sudo ufo allow out on tun0

(If at any point your internet stops working you can do ```sudo ufw reset``` to get you out of the mess.)

Let's break this down. ```ufw enable``` switches ufw on, simple enough.

```ufw reject outgoing``` says you want to stop ALL outgoing traffic by default. After this we need to add some exceptions to this default rule...

```ufo allow out on 53/udp``` is to allow DNS traffic (DNS uses port 53). This isn't strictly necessary but some VPN providers don't route DNS requests through the VPN. (Paranoid people might want to ensure they pick a provider that does tunnel DNS?).


```ufo allow out on 1194/udp``` is to allow openVPN to connect on its regular UDP port - this is the one thing that is permitted to use your regular internet connection.

```udo allow out on tun0``` this final command is to allow any data going through the tunnel.

Once everything is working you should be able to verify it like so:

    $ sudo ufw status verbose
    Status: active
    Logging: on (low)
    Default: deny (incoming), reject (outgoing), disabled (routed)
    New profiles: skip
    
    To                         Action      From
    --                         ------      ----
    53/udp                     ALLOW OUT   Anywhere
    1194/udp                   ALLOW OUT   Anywhere
    Anywhere                   ALLOW OUT   Anywhere on tun0
    53/udp (v6)                ALLOW OUT   Anywhere (v6)
    1194/udp (v6)              ALLOW OUT   Anywhere (v6)
    Anywhere (v6)              ALLOW OUT   Anywhere (v6) on tun0

There you have it! Traffic is only allowed through the VPN tunnel except DNS and the tunnel itself! You can test this by killing the tunnel and check that you can't load any webpages etc.

You could make this tighter, for example by allowing DNS / Tunnel traffic to specific hosts, or by tunneling the DNS requests. Check the UFW man pages.

Let me know if this has been useful :)

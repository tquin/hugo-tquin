---
title: "The ✨ Magic ✨ of Tailscale"
date: 2024-01-06
draft: false
tags:
  - tailscale
  - vpn
  - network
---

Have you heard the good news? About peer-to-peer virtual mesh networking?

<!--more-->

{{< figure src="/posts/magic_of_tailscale/saved.webp" class="smaller_img" alt="img_saved" >}}

At this point, everyone's familiar with how VPN servers work -- you send traffic to the VPN, and it forwards traffic onwards _somewhere._ Historically, that would usually be to let you access a private network you normally couldn't, like a home or work LAN. Nowdays, it's also common to route Internet traffic through a VPN, not for increasing access but for some level of geographic or privacy misdirection. This post is focused on the first type, but this tool can do many ✨ magical ✨ things beyond just what I've personally used it for.

---

## So what is this thing?

[Tailscale][link_tailscale] creates virtual networks that can let machines talk to each other even when they don't have a direct connection. The mechanics behind Oz's curtain are Wireguard keypairs, firewall-circumventing trickery, and NAT-evading subterfuge, but the reason for the title of this post is that it all happens behind the scenes, and things [Just Work™][link_it_just_works] ✨ magically ✨ to the user.

These two [blog][link_how_it_works] [posts][link_nat_traversal] from the company explain the inner cogs and gears if you're interested, but I'd like to go through some of the cool things I've found useful with Tailscale in my own networking adventures.

---

## SSH from a World Away

The first use I had for this sort of tech was SSH. Often, I'm not at home, but I want to quickly restart something running on my home server.

The most obvious answer to this would be "open port 22 to the internet!", which as soon as you try, will make you realise [quite how bad an idea][link_honeypot] that is. The second thought would be some level of indirection in this process, like first authenticating to a reverse proxy before you can hit the actual SSH server. But the core problem still exists - anyone can spam whatever endpoint you have exposed until they crack it, saturate your bandwidth, or get your innocent VPS locked down by your hosting provider for its own good.

The direct connections between nodes enabled by Tailscale mean all these issues just disappear. Alongside their primary `eth0` or Wifi network interfaces, every host running the client software creates a new interface in the imaginary virtual world (called a "tailnet"). This is much the same as how running a VM or Docker container will usually result in a new interface to bridge between the host and its virtual instances.

 These are in a range reserved for CG-NAT, so unlikely to conflict with anything you might have on a usual LAN like `10.*` or `192.168.*`. For example, say I want to connect from my laptop to my home server while I'm out and about. Well, they can both communicate directly through the tailnet, bypassing any firewall configuration on my home router that would usually block the packet. I'll say it again - it's like ✨ magic! ✨

> {{< figure src="/posts/magic_of_tailscale/direct_ssh.png" alt="img_direct_ssh" >}}
> _Recognise the naming scheme?_

---

## Memorable DNS

While the above section does functionally fulfill my requirement, it's not the most user-friendly. I'm not going to remember the additional IP given to all of my machines in the tailnet, and luckily I don't have to. Tailscale also includes ✨ MagicDNS ✨ _(I swear that's their name for it, not mine)_ for facilitating connectivity.

Every node registers its hostname under your unqiue tailnet's DNS (like `my-host.isnt-this-fun.ts.net`), and in most cases, you don't even need to specify the full FQDN. This means that I can literally just type `ssh bombe`, and it works. No username needed since it's the same on both ends, no password prompt, no `-i keyfile` argument to remember. It could not be simplier, but it's still just as secure using Wireguard public/private keys under the hood. You can even do this outside a CLI, like in the browser - navigate to `bombe:80` in Firefox and I could hit a locally hosted webpage from anywhere on Earth.

---

## ACLs, NACLs, RBAC, and more Acronyms

Despite how easy this is, it's not a laissez-faire attitude to authorization. You can define controls to be as specific as you like for the different services you're using. Maybe some subnets can only talk to each other one-way, like syncing prod to preprod but not the other way around. You can tag machines and groups, like maybe only my `favourite` people can connect to my `special` machines. There's even some built-in smart "autogroups", like the `self` group to say everyone can SSH to their own set of machines, but not each other's.

---

## Subnet Routing

Picture this: you've finally updated an ancient Haswell-era server with a new CPU and motherboard, yay! Satisfied that everything's still running fine and working after rebooting into the OS, I left the house and thought no one would be any the wiser. But oh no, disaster struck. Even through I tested the Plex server I'm running still worked fine on the LAN, I can't access it anymore from outside. What's the problem? I'll give you a second to think\...

{{< figure src="/posts/magic_of_tailscale/countdown.gif" class="smaller_img" alt="img_countdown" >}}

New motherboard, new MAC address baby! My home router, seeing this as a new device, gave it a completely different DHCP lease than it had previously. Those port forwarding rules I had set up for Plex (`32400`) and Minecraft (`25565`) are now completely useless to me, and I can't access the router's settings to fix it until I get home. It's not like I can install Tailscale on an appliance like a consumer router to manage it remotely.

Enter the solution: subnet routing. I can reach the home server itself through Tailscale, and it obviously can see my home LAN. Subnet Routing lets me set up that Tailscale client to be **a VPN server!** All the server has to do is advertise "hey, I can see `192.168.1.1/24` subnet from here", and my laptop can forward packets to the server through Tailscale so the server can forward them along to the router, just one more hop in the chain of connections.

Of course, like everything else, it works via ✨ magic ✨ - just typing `http://192.168.1.1` connects me straight through to the router's web UI. This same system could let you access printers or IoT devices that can't run the Tailscale client natively, just by setting up something else to bridge the two.

---

## FUN with FUNnels

Out of everything on this list, it's the only one I haven't personally used myself (yet). But it's just such a cool concept, that I can't help but also mention it. This comes in two separate but related parts: Funnel lets you expose a locally-running service publicly to the whole internet; Serve is its counterpart that keeps sharing private to your little tailnet.

This lets you run a website, serve a file download, or host a game server that's accessible to anyone, but due to that ✨ same word ✨ I keep mentioning, not have to expose your own public IP address in the process. And even if I pick up my laptop and physically move to a different network, it doesn't matter; as soon as you reconnect to the tailnet and advertise your new IP, Funnel/Serve traffic can still route its way to you via the proxy.

---

## Commerical, but Open Source

As much as I've praised it here, Tailscale isn't the only solution out there for this sort of thing. [ZeroTier][link_zerotier] is another option that solves many of the same problems, but fundamentally approaches it from a Layer 2 (eg. Ethernet) rather than a Layer 3 (eg. IP) approach. I suspect either would work for my needs, and in these sorts of situations it's best to just pick something and start using it rather than flail in decision paralysis.

Ultimately for me, I feel a little more confident in choosing a BSD-licensed software stack than a BSL-based one, but software licensing is a hot debate I'm not brave enough to venture into right now. Despite both of these tools being company- rather than community-led, both are open-source and you can even [self-host the management plane][link_headscale] for both, so I feel optimistic incorporating it into my personal workflow.

---

There's so much more in this ✨ magical ✨ topic I'd like to cover. Did you know you can share an individual node between different tailnets? Or setup a node as an "exit node" and route other machine's internet traffic through your own private VPN? Or define split tunnelling rules without the eternal nightmare that is messing with `iptables` config files? But, this is already many more words on this topic than I thought I'd write, so I'd probably best wrap it up here for now.

I really think it's worth a try if you have more than one machine you use, particularly if one of them is a server that you'll leave running when you're not there. Give it a go!




[link_tailscale]: https://tailscale.com/
[link_how_it_works]: https://tailscale.com/blog/how-tailscale-works
[link_nat_traversal]: https://tailscale.com/blog/how-nat-traversal-works
[link_it_just_works]: https://youtu.be/qmPq00jelpc?si=P8lV8XEwxLskoDMI
[link_honeypot]: https://securehoney.net/
[link_zerotier]: https://www.zerotier.com/
[link_headscale]: https://github.com/juanfont/headscale

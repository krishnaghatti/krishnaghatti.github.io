---
layout: post
title:  "Pi-hole in the cloud: Part 1"
date:   2020-04-07 00:00:00 +0530
categories: jekyll update
---

`Pi-hole` is a DNS level ad blocker. Once up and set as the DNS server of the local network, it blocks out all the DNS queries from the client that are trying to resolve any domains that server ads. All this is done without installing any software on the client side.
The name Pi-hole comes from the fact that this is usually run as a service on a Raspberry Pi a small all-in-one PC.

I have a Raspberry-pi that I used to setup Pi-hole at my home. I added the Pi node as my DNS server in my router for my local network. This blocks all the ads on all the devices on my network and also give me a view of each client activity(if enabled). The setup works great on clients connected over wi-fi or ethernet to the router. But how to block the ads and unwanted domains when on mobile network?

I can run the pi-hole service on a public cloud and use that as a resolver on my phone. But I will end up setting up a DNS server in the public network and if left open can be used for [DNS amplification attacks][dns-amp] and also I will be incurring a cost for the amount of network ingress the node gets which I am not willing to pay at this moment.

To over come this I used [Wireguard VPN][wireguard] server on the cloud node. I connected my mobile to the cloud node's network via a dedicated VPN. This avoided the need of:
1. Having a public IP on the DNS server
2. Removes the challenge of blocking unwanted queries.

As of today I have been using this setup for over 10 days and I am very happy with the outcome. Below is a screenshot of the stats over a 24 hour period and **34.7% of my DNS queries on my mobile are related to ads!!**

![Stats over 24 hours](/assets/images/piholestats.jpeg)

I will be detailing the technical aspects along with the configuration in a following post.

<!-- The setup can be done with [OpenVPN server][pi-vpn] -->
Do check out the excellent documentation available on [Pi-hole docs][pi-hole] for more information on how to get the most out of Pi-hole. File all bugs/feature requests at [Pi-hole’s GitHub repo][pihole-gh]. If you have questions, you can ask them on [Pi-hole discourse][pi-hole-disc].

[pi-hole]: https://docs.pi-hole.net/
[pihole-gh]:   https://github.com/pi-hole/pi-hole/issues
[pi-hole-disc]: https://discourse.pi-hole.net/
[dns-amp]: https://www.cloudflare.com/learning/ddos/dns-amplification-ddos-attack/
<!-- [pi-vpn]: https://docs.pi-hole.net/guides/vpn/only-dns-via-vpn/ -->
[wireguard]: https://en.wikipedia.org/wiki/WireGuard

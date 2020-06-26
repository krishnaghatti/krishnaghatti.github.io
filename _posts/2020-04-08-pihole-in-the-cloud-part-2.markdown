---
layout: post
title:  "Pi-hole + cloud: Part 2 - Cloud VM and WireGuard setup"
date:   2020-04-08 00:00:00 +0530
categories: jekyll update
---

In [Part-1][part1] I talked about why I wanted this setup. In Part-2 I will be covering the setup details that are required to run WireGuard and a VM on which to run it.

- ### A cloud VM
  The first ingredient is a VM. I am using [Digital Ocean][do] droplet for this purpose. I picked a Ubuntu 18.04 droplet with 1 GB Memory and 25 GB Disk. The droplet has about **1 TB** in network transfer usage per month which is a lot for this requirement. I can run a few more services on the same machine later. This will cost around $5 to $6.

  I chose Digital Ocean's Bangalore datacenter. As I want to use this machine for my DNS queries, I want it to be closest to my client as possible. To check the network latency I did a ping test with Google DNS server 8.8.8.8 and my server.

      $ ping -c 3 8.8.8.8
      PING 8.8.8.8 (8.8.8.8): 56 data bytes
      64 bytes from 8.8.8.8: icmp_seq=0 ttl=56 time=16.004 ms
      64 bytes from 8.8.8.8: icmp_seq=1 ttl=56 time=14.263 ms
      64 bytes from 8.8.8.8: icmp_seq=2 ttl=56 time=13.957 ms

      --- 8.8.8.8 ping statistics ---
      3 packets transmitted, 3 packets received, 0.0% packet loss
      round-trip min/avg/max/stddev = 13.957/14.832/16.004/0.714 ms

      $ ping -c 3 16.22.21.21
      PING 16.22.21.21 (16.22.21.21): 56 data bytes
      64 bytes from 16.22.21.21: icmp_seq=0 ttl=55 time=14.650 ms
      64 bytes from 16.22.21.21: icmp_seq=1 ttl=55 time=12.282 ms
      64 bytes from 16.22.21.21: icmp_seq=2 ttl=55 time=11.852 ms

      --- 16.22.21.21 ping statistics ---
      3 packets transmitted, 3 packets received, 0.0% packet loss
      round-trip min/avg/max/stddev = 11.852/12.669/14.650/1.022 ms

  In a few tests I did the average ping latency is almost same as 8.8.8.8. That is promising. Let's move to the next step.

  IP packets should be allowed to be forwarded from this VM. This is done by uncommenting  the line `net.ipv4.ip_forward=1` in the file `/etc/sysctl.conf` and run `sysctl -p` for the changes to take effect without requiring a reboot.

      # sysctl -p
      net.ipv4.ip_forward = 1
      # cat /proc/sys/net/ipv4/ip_forward
      1

- ### [Wireguard VPN][wireguard] setup
  WireGuard is a relatively new VPN software but it is the one of the very few projects that can boast of being merged into the Linux kernel. It is available by default from kernel version 5.6. And in the words of the big man himself...

  >Can I just once again state my love for [WireGuard] and hope it gets merged soon?
  Maybe the code isn't perfect, but I've skimmed it, and compared to the horrors that are
  OpenVPN and IPSec, it's a work of art - Linus Torvalds

  A couple of best parts of WireGuard which are very much required in my use case.
  - Establishes connections in less than 100ms. That is wicked fast!
  - IP roaming support - meaning I can change wifi networks or disconnect from wifi or cellular and the VPN tunnel connection won't be lost. It just works!

  The required packages are in the official ppa repo. Let's add that to our VM and install WireGuard.

      $ sudo add-apt-repository ppa:wireguard/wireguard
      $ sudo apt-get update
      $ sudo apt-get install wireguard-tools wireguard-dkms
  I will switch to root shell and run `wg` to check if installation was successful which should not output anything if everything is OK.

      $ sudo -s
      # wg
      #

  The next step is to generate the required public and private keys. I will keep them under `/etc/wireguard/keys` directory. Also I will change the permissions to allow only root to change these files.

      # mkdir /etc/wireguard/keys
      # cd /etc/wireguard/keys
      # umask 077

  WireGuard uses asymmetric public/private Curve25519 key pairs for authentication between client and server. I will generate both the private and public key at once by piping the private key output to tee to save it to the file `privatekey` and also to forward the private key to `wg publickey` which derives the public key from `privatekey` and the save it to `publickey`.

      # wg genkey | tee privatekey | wg pubkey > publickey
      # ls
        privatekey  publickey

  The configuration of WireGuard is in `wg0.conf`.

      [Interface]
      Address = 192.168.5.1/24
      ListenPort = 51820
      PrivateKey = QPvmG/xa9xPvD5XV5K5pyXeA+6x8aiCiRC82RECffGY=


  - Address = 192.168.8.1/24. This line defines the IP and the subnet the server will be running on. This needs to be a network that is not being used by the cloud VM so that it does not interfere with the routes of the VM.
  - ListenPort = 51820. This is the listening port of the VPN server. Make sure that the VM has this port open to accept UDP traffic from any where (0.0.0.0/0).
  - PrivateKey is the key that was generated in the previous step.

  Run the command `wg-quick up wg0` to start the VPN server.

      # wg-quick up wg0
      [#] ip link add wg0 type wireguard
      [#] wg setconf wg0 /dev/fd/63
      [#] ip -4 address add 10.0.0.1/24 dev wg0
      [#] ip link set mtu 8921 up dev wg0

  To check the status of the VPN tunnel run the command `wg show`

      # wg show
      interface: wg0
      public key: vgBavl4f13wT2qbWj0OgLbDNxhls+nrjWPYngmXaIT8=
      private key: (hidden)
      listening port: 51820

In the next parts I will be walking through the Pi-hole setup, client configuration and validating and testing the setup.

[Pihole]: https://pi-hole.net/
[wireguard]: https://en.wikipedia.org/wiki/WireGuard
[part1]: 2020-04-07-pihole-in-the-cloud-part-1.markdown
[do]: https://cloud.digitalocean.com/

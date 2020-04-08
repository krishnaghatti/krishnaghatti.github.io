---
layout: post
title:  "Pi-hole + cloud: Part 3 - Pi-hole and Client configuration"
date:   2020-04-08 00:00:00 +0530
categories: jekyll update
---

In [Part-1][part1] I talked about why I wanted this setup. [Part-2][part2] talks about the setup of the VM and the WireGuard server. Now I will talk about the main ingredient the Pi-hole setup along with the client configuration.

- ### Pi-hole Setup

  The prerequisites are simple.
  - [x] It requires 512mb or ram, and about 50mb of free space.
  - [x] A supported operating system.

  With prerequisites checked, the installation is also simple. Just run the one step installer that takes care of all the configuration required. One thing to remember is to select the `wg0` interface that we created for WireGuard VPN server as the network interface for Pi-hole.

      # curl -sSL https://install.pi-hole.net | bash

- ### Client Configuration

  Similar to the public/private keys generated for the VPN server, each client also need to have a key pair. The `publickey` of each client should be shared with the server and the `publickey` of the server will be shared with the client.

      # wg genkey | tee cl_privatekey | wg pubkey > cl_publickey
      # ls
        cl_publickey cl_privatekey

  The `publickey` of each client(my mobile phone in my case) needs to be added to the server configuration along with the IP address that the client will have when connected to the VPN server. The server configuration that is the `wg0.conf` will effectively be:

      [Interface]
      Address = 192.168.5.1/24
      ListenPort = 51820
      PrivateKey = QPvmG/xa9xPvD5XV5K5pyXeA+6x8aiCiRC82RECffGY=

      [Peer]
      PublicKey = /cPFo6mwfIG3XYTJU1JeutsGzWLINdTm9NOQUriNLlg=
      AllowedIPs = 192.168.5.5/32

  Where `PublicKey` is the public key of the client generated in the previous step and `AllowedIPs` is the IP that I want this client to have when on VPN.

  The client configuration file will also look similar.

      [Interface]
      Address = 192.168.5.5/32
      ListenPort = 51820
      PrivateKey = gEPGRLrDBiolZ/5P4G5bj2aq37lSwCw8ebN2P8gUwlI=
      DNS = 192.168.5.1

      [Peer]
      Publickey = vgBavl4f13wT2qbWj0OgLbDNxhls+nrjWPYngmXaIT8=
      AllowedIPs = 0.0.0.0/0, ::/0
      Endpoint = 16.22.21.21:51820

  In the client configuration, the `PrivateKey` is the private key generated for the client in the previous step. The `DNS` is the IP of the `wg0` interface where Pi-hole is listening for DNS queries. The `Endpoint` is the public IP of my VPN server along with the port on which the VPN service is listening for connections. As all UDP traffic is allowed on my
  The `AllowedIPs` in the client has to be mentioned specially. If I set it to `0.0.0.0/0, ::/0` like above, all the traffic from the client will be forwarded to the VPN server. This means if I open a video on YouTube on my phone, the request is made to the VPN server which in turn gets the content from YouTube and serves the client. Simply put all the traffic flows through the VPN server.

  If `AllowedIPs` is set to only the private VPN IPs (192.168.5.1/32 and 192.168.5.5/32 in my case) only the DNS traffic flows though. So on my phone all the DNS queries go the Pi-hole IP and the subsequent requests use the WiFi or the cellular network. This is exactly what I want.

  On the client i.e, my mobile phone, I installed WireGuard client. The client configuration file can be directly imported by clicking on the `+` icon in the WireGuard app. Once imported just connect the client and the `key` symbol comes up on the top notification area showing that the VPN tunnel is established. Below is a screenshot of both these steps.

  ![Import client config and connect to VPN](/assets/images/vpn_connected.png)

  On the VPN server we can check the status with `wg show` command that shows the connected peers and the last time the handshake happened along with the data transfer for this peer.

      interface: wg0
        public key: vgBavl4f13wT2qbWj0OgLbDNxhls+nrjWPYngmXaIT8=
        private key: (hidden)
        listening port: 51820

      peer: /cPFo6mwfIG3XYTJU1JeutsGzWLINdTm9NOQUriNLlg=
        endpoint: 18.19.14.29:18540
        allowed ips: 192.168.5.5/32
        latest handshake: 19 seconds ago
        transfer: 2.79 MiB received, 4.64 MiB sent

  This concludes the configuration required to setup the VPN server, Pi-hole instance and the client setup to go along with the server.

  voilà! Now my mobile phone where ever it is will have no ads served. The list of the blocked domains are periodically updated and I can add any wildcard/regex pattern domains to add more to the list. In my next post I will show this setup in action.

[part1]: https://blog.krishnaghatti.dev/jekyll/update/2020/04/07/pihole-in-the-cloud-part-1.html
[part2]: https://blog.krishnaghatti.dev/jekyll/update/2020/04/08/pihole-in-the-cloud-part-2.html

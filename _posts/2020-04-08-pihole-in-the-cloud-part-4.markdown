---
layout: post
title:  "Pi-hole + cloud: Part 4 - Testing"
date:   2020-04-08 00:00:00 +0530
categories: jekyll update
---

In [Part-1](pihole-in-the-cloud-part-1.markdown) I talked about why I wanted this setup. [Part2](2020-04-08-pihole-in-the-cloud-part-2.markdown) talks about the setup of the VM and the WireGuard server.In [Part3](2020-04-08-pihole-in-the-cloud-part-3.markdown) covered the Pi-hole and the client configurations let's see the setup in action and if the DNS server actually can perform similar to a open DNS server like 8.8.8.8.

- ### Ad blocking

The screenshot below is a side by side comparison with and with out the VPN tunnel connected. When the tunnel is switched off, the DNS queries go to the ISP DNS server which gives the actual ad content. When the tunnel is up, the Pi-hole DNS server responds that the ad is available at 0.0.0.0 thus the client gets a null response which effectively blocks the ad.

![Ads blocked comparison](/assets/images/ad-block-comparision.jpg)


![B E A Utiful](/assets/images/beautiful.gif)

- ### Testing
  Let me check if the actual DNS response time is comparable to a Google DNS server.

      $ dig youtube.com @192.168.5.1

      ; <<>> DiG 9.14.3 <<>> youtube.com @192.168.5.1
      ;; global options: +cmd
      ;; Got answer:
      ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42303
      ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

      ;; OPT PSEUDOSECTION:
      ; EDNS: version: 0, flags:; udp: 1452
      ;; QUESTION SECTION:
      ;youtube.com.                   IN      A

      ;; ANSWER SECTION:
      youtube.com.            274     IN      A       216.58.199.174

      ;; Query time: 50 msec
      ;; SERVER: 192.168.5.1#53(192.168.5.1)
      ;; WHEN: Tue Apr 07 23:23:54 IST 2020
      ;; MSG SIZE  rcvd: 67


      $ dig youtube.com @8.8.8.8

      ; <<>> DiG 9.14.3 <<>> youtube.com @8.8.8.8
      ;; global options: +cmd
      ;; Got answer:
      ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14855
      ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

      ;; OPT PSEUDOSECTION:
      ; EDNS: version: 0, flags:; udp: 512
      ;; QUESTION SECTION:
      ;youtube.com.                   IN      A

      ;; ANSWER SECTION:
      youtube.com.            299     IN      A       172.217.166.110

      ;; Query time: 118 msec
      ;; SERVER: 8.8.8.8#53(8.8.8.8)
      ;; WHEN: Tue Apr 07 23:00:15 IST 2020
      ;; MSG SIZE  rcvd: 56

  The DNS query time when using the Pi-hole as my DNS server is around 50ms compared to 118ms when using 8.8.8.8. I can live with that :stuck_out_tongue_winking_eye:.

  OK Let me not be too carried away. Unlike the open DNS servers, my server only serves 1 client and also has a little cache which makes the frequent queries resolve a little bit faster. Some day I will run dnsperf tests and see if this tiny server can take more load.

- ## Challenges

  While the setup works as per my expectation and the results are impressive, there are some challenges I have to resolve at some point.
    - The server is in the Bangalore region and so as long as I use it in and around that area I should have no latency issues. I have not tested this when the client is in other parts of the world.

- ### Automation

  I setup the cloud VM with Terraform and the configuration with Ansible. The [Algo][algo] project has a full setup that provisions VM, setups up pi-hole, wireguard and other VPN servers to choose from. Do check it out if you want to have the full setup automated.

[algo]: https://github.com/trailofbits/algo

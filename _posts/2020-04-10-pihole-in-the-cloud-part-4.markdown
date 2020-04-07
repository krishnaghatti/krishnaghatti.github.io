---
layout: post
title:  "Pi-hole in the cloud: Part 4 - Testing the setup"
date:   2020-04-10 00:00:00 +0530
categories: jekyll update
---

In [Part-1][part1] I talked about why I wanted this setup. In Part-2 I will be covering the setup details that are required to run WireGuard and a VM on which to run it.

- ### Testing
  Let me check if the actual DNS response time is comparable to a Google DNS server. I will be using the `time` command to check the time for each query.

      $ time dig youtube.com @192.168.8.1

      ; <<>> DiG 9.14.3 <<>> youtube.com @192.168.8.1
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
      ;; SERVER: 192.168.8.1#53(192.168.8.1)
      ;; WHEN: Tue Apr 07 23:23:54 IST 2020
      ;; MSG SIZE  rcvd: 67

      real    0m0.092s
      user    0m0.016s
      sys     0m0.026s

      $ time dig youtube.com @8.8.8.8

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

      real    0m0.160s
      user    0m0.001s
      sys     0m0.041s

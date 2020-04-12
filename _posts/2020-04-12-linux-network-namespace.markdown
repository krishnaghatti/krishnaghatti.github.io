---
layout: post
title:  "Network namespaces"
date:   2020-04-12 00:00:00 +0530
categories: jekyll update
---

Namespaces are one of the fundamental building blocks for containers. According to [Wikipedia][ns] a namespace is:

>a feature of the Linux kernel that partitions kernel resources such that one set of processes sees one set of resources while another set of processes sees a different set of resources.

There are 8 types of namespaces in the Linux kernel (a new `time` namespace was recently added to the kernel source. So not all versions of the kernel will have this). You can use `lsns` command to list all the namespaces in the system. In this post I will share my understanding of the use-case of `network` namespace.

I will be creating a virtual netowork as below. I will be explaining the details along the way. Please remember that all this happens with in a single system.

    +----------------+                                +----------------+
    |                |                                |                |
    |                |                       +--------+                |
    |                |   +-------------------+VETH1   |                |
    |               ++-------+               |10.1.1.1|                |
    |               |        |               |        |                |
    |   DEFAULT     | VETH0  |               +--------+    BLUE        |
    |   NAMESPACE   | 10.1.1.2                        |    NAMESPACE   |
    |               ++---+---+               +--------+                |
    |                |   |                   |VETH3   |                |
    |                |   +-------------------+10.1.1.3|                |
    |                |                       |        |                |
    |                |                       +--------+                |
    +----------------+                                +----------------+


A new linux system has no other namespaces except for the default one. All the physical interfaces of the system belong to this. The `ip` command is only one I will need  to play with the network namespaces. `ip link list` and `ip netns list` will list the interfaces and network namespaces respectively.

    # ip link list
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group  default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode    DEFAULT group default qlen 1000
        link/ether 9a:8d:09:82:8f:ba brd ff:ff:ff:ff:ff:ff

    # ip netns list
    #

The second command output shows nothing because there are no namespaces. Let me create one.

    # ip netns add blue
    # ip netns list
    blue (id: 0)

I will create a `veth pair` with the below command. The `veth` interfaces come in pairs. Each of which can be attached to different namespaces to connect them (I will show what it means in the next steps).

    # ip link add veth0 type veth peer name veth1
    # ip link list
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
        link/ether 9a:8d:09:82:8f:ba brd ff:ff:ff:ff:ff:ff
      8: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
        link/ether f2:cf:0e:6f:d9:5b brd ff:ff:ff:ff:ff:ff
      9: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
        link/ether b2:1d:21:8d:da:26 brd ff:ff:ff:ff:ff:ff

By default any new virtual interfaces created are added to the default namespace unless explicitly specified. I will move one of the `veth` interface pair to the `blue` namespace I created.

      # ip link set veth1 netns blue
      # ip link list
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
          link/ether 9a:8d:09:82:8f:ba brd ff:ff:ff:ff:ff:ff
      9: veth0@if8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
          link/ether b2:1d:21:8d:da:26 brd ff:ff:ff:ff:ff:ff link-netnsid 0

Now when I ran the `ip link list` command, the `veth1` is no longer in the default namespace and so the output only shows `veth0` along with the `lo` and `eth0` interfaces.

To run any commands inside a namespace, I have to run `exec` inside that namespace like below.

      # ip netns exec blue ip link list
      1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      8: veth1@if9: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT groudefault qlen 1000
          link/ether f2:cf:0e:6f:d9:5b brd ff:ff:ff:ff:ff:ff link-netnsid 0

The output only shows the interfaces in the `blue` namespace and not the ones from the  default one. Let me assign an ipaddress to the interface in the `blue` namespace and bring it `UP`. Like before to see the `ip a` details inside the namespace I have to `exec` into it.

      # ip netns exec blue ip addr add 10.1.1.1/24 dev veth1
      # ip netns exec blue ip a
      1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      8: veth1@if9: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
          link/ether f2:cf:0e:6f:d9:5b brd ff:ff:ff:ff:ff:ff link-netnsid 0
          inet 10.1.1.1/24 scope global veth1
          valid_lft forever preferred_lft forever
      # ip netns exec blue ip link set dev veth1 up

I will create another veth pair and add one of the interface into the `blue` namespace and  this interface will have the ip `10.1.1.3`. Also I will assign the IP `10.1.1.2` to the `veth0` interface in the default namespace but will not bring it UP.

      # ip link add veth2 type veth peer name veth3
      # ip link set veth3 netns blue
      # ip netns exec blue ip addr add 10.1.1.3/24 dev veth3
      # ip netns exec blue ip link set dev veth3 up
      # ip addr add 10.1.1.2/24 dev veth0

Now the interfaces `veth1` and `veth3` are in the same namespace. Also they are in the same network subnet and so should be reachable.

      # ip netns exec blue ping 10.1.1.3
      PING 10.1.1.3 (10.1.1.3) 56(84) bytes of data.
      64 bytes from 10.1.1.3: icmp_seq=1 ttl=64 time=0.020 ms

      # ip netns exec blue ping 10.1.1.1
      PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
      64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=0.020 ms

      # ping 10.1.1.1
      PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
      ^C
      --- 10.1.1.1 ping statistics ---
      4 packets transmitted, 0 received, 100% packet loss, time 3068ms

This is as good as having 2 systems in the same subnet. If these interfaces are assigned to any VM or a container, they will be able talk to each other. But when I ping the IP `10.1.1.1` from the default namespace, it is not reachable as there is no connection or `route` between these two networks.

In the next step I will bring UP the `veth0` interface. This basically connects the default and the `blue` namesapces. Once the link is UP, we can reach the `veth1` IP from the default namespace and `veth0` from the `blue` namespace.

      # ip link set dev veth0 up
      # ping 10.1.1.1
      PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
      64 bytes from 10.1.1.1: icmp_seq=1 ttl=64 time=0.025 ms

      # ip netns exec blue ping 10.1.1.2
      PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
      64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=0.023 ms

      # ip netns exec blue ip route
      10.1.1.0/24 dev veth1 proto kernel scope link src 10.1.1.1
      # ip route
      default via 134.209.144.1 dev eth0 proto static
      10.1.1.0/24 dev veth0 proto kernel scope link src 10.1.1.2

Great! But what is the use case of this? Let's see that in a TLDR fashion.

When only the interfaces inside the `blue` namespace are up they were able to talk to each other but not to the outside world. This is how a completely `private network` can be created. Once the interface in the default namespace came up, we were able to reach the host network and this can be used to create `host only` networks i.e, systems that can only work inside that host as the networks and the interfaces are confined to a single host.

Hope that gave some insights into the network namespace. In my future posts I will be exploring other kind of namesapces and see how all of these collectively make container's what they are and how the isolation of resources work.

[ns]: https://en.wikipedia.org/wiki/Linux_namespaces

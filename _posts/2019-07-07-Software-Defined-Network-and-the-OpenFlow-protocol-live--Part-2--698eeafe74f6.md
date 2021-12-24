---
layout: post
title: Software Defined Network and the OpenFlow protocol live (Part 2)
categories: [sdn]
excerpt: "Simple implementation of a switching hub."
---

_Originally published on medium_


![](https://cdn-images-1.medium.com/max/2560/0*-SGR1ru7IGFnZ5iv.png)

This post contains a simple implementation of a switching hub. [Part
1]({% link _posts/2019-07-07-Software-Defined-Network-and-the-OpenFlow-protocol-live--Part-1--8bbbbb42242e.md %}) of this post contains a quick theoric overview.


### Set Up

Let's get into practice.

-   First, set up a **virtual machine**:

I know this can be very tiresome but the tools we are going to install
may overwrite some files in your home directory, so avoid the stress and
set up a Ubuntu virtual machine.

-   Install [Mininet](http://mininet.org/)

Mininet is a system for emulating a network with particular emphasis on
SDNs. Mininet allows one to create a virtual network containing hosts,
switches, links, and controllers.

```bash
sudo apt-get install mininet
```

The Mininet will be used to set a test network architecture.

-   Install [Ryu](https://osrg.github.io/ryu/)

> A Ryu provides software components with well-defined API that make it
> easy for developers to create new network management and control
> applications. Ryu supports various protocols for managing network
> devices, such as
> [OpenFlow](https://www.opennetworking.org/)

```bash
sudo apt install python-pip
pip install ryu
```

The Ryu will be used to set the controller.

### SDN Example --- Switching Hub

To exemplify an SDN we are going to build a **Switching Hub.** A switch
by itself resembles a network hub. Unlike hubs, however, network
switches are capable of inspecting incoming messages as they are
received and directing them to a specific communications port:

![Switch](https://cdn-images-1.medium.com/max/800/1*3JBMvfjU6frwnMAu7--k1A.gif)

However, we are implementing a **Switching Hub**, which is a switch that
floods all other hosts/devices for the first time that the packet
arrives:

![Hub](https://cdn-images-1.medium.com/max/800/1*pf_ZBPoMAiFmVI3sta_E0g.gif)

The switching hub has the following functions:

1.  Learns the MAC address of the host connected to a port and retains
    it in the MAC address table.
2.  When receiving packets addressed to a host already learned,
    transfers them to the port connected to the host.
3.  When receiving packets addressed to an unknown host, performs
    flooding.

The switch analyzes the received packets to learn the MAC address of the
host and information about the connected port. That being said, when the
switch receives a packet from an unknown it acts as a hub, however, if
the switch already has forwarded packets from that host once before it
will act as a switch.

The learning part is the responsibility of the controller. To achieve
this behavior we will use the Ryu implementation of the simple switch.
This sample code can be used as a boilerplate to build any other switch
variation.

The source code of the switching hub is in Ryu's source tree:

> *ryu/app/example_switch_13.py*

### Environment Building

-   [First, build an environment on Mininet:]{#2a45}

```bash
sudo mn --topo single,3 --mac --controller remote --switch ovsk
```

Here's the output of this command:

![](https://cdn-images-1.medium.com/max/800/1*WTYNAODep1Y4cO2srQJ8qQ.png)

We just created a realistic virtual network with one switch, one
controller, and three hosts.

-   [Let's start an xterm for the controller and for the switch.]{#9849}

```bash
xterm c0 s1
```

![](https://cdn-images-1.medium.com/max/800/1*yfNQRp7AnDOzhD9A1YJt6w.png)

-   Next, set the version of OpenFlow to be used on the switch to
    1.3:

```bash
ovs-vsctl set bridge s1 protocols=OpenFlow13
```

![](https://cdn-images-1.medium.com/max/800/1*1LLDDnfV7leeaEjb8f1Zmg.png)

-   Then, start example_switch_13 on the xterm of the
    controller:
```bash
ryu-manager --verbose ryu.app.example_switch_13
```

![](https://cdn-images-1.medium.com/max/800/1*nvvuVEA5-QYe2DnuCHTuNw.png)

Now OVS has been connected, the handshake has been performed, the
Table-miss flow entry has been added and the switching hub is in the
status waiting for Packet-In.

-   Ping from host 1 to host 2:

```bash
h1 ping -c1 h2
```

![](https://cdn-images-1.medium.com/max/800/1*1_hzxp_-2zE0sabAwG9FGg.png)

-   Now, let's check the flow table:

```bash
ovs-ofctl -O openflow13 dump-flows s1
```

![](https://cdn-images-1.medium.com/max/800/1*Z_Ldpg69TTONSgZh1IEeow.png)

Two flow entries of priority level 1 have been registered.

1.  Receive port (in_port):2, Destination MAC address (dl_dst):host 1
    -\> Action (actions):Transfer to port 1
2.  Receive port (in_port):1, Destination MAC address (dl_dst): host 2
    -\> Action (actions): Transfer to port 2

-   Checking the log output of the controller:

![](https://cdn-images-1.medium.com/max/800/1*NACdCwPjKnpmiva-XYg6pw.png)

-   Running the ping again and checking the log output of the
    controller again:

```bash
h1 ping -c1 h2
```

![](https://cdn-images-1.medium.com/max/800/1*dRf9UlX5x0YIDGQKVUXljg.png)

Note that the packets don't pass through the controller on the second
time, showing that the switch learned the packet destination and
addressed to the correct host.

**\*OBS**: The switch keeps receiving packets to establish which hosts
are ready to communicate, you should ignore these events on the
controller log.

### Conclusion

We just used the default implementation of a simple switching hub to see
how to build a Software Defined Network with a Ryu application running
as a controller and using the OpenFlow protocol to control the switch.

### References

-   [Switching
    Hub](https://osrg.github.io/ryu-book/en/html/switching_hub.html#switching-hub)
-   [RYU Controller
    Tutorial](http://sdnhub.org/tutorials/ryu/)
-   [Firewall](https://osrg.github.io/ryu-book/en/html/rest_firewall.html)
-   [Mininet](http://mininet.org/)

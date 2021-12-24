---
layout: post
title: Software Defined Network and the OpenFlow protocol live (Part 1)
categories: [sdn]
excerpt: "Quick literature overview around
SDN (Software Defined Network) and the OpenFlow protocol"
---

_Originally published on medium_


![](https://cdn-images-1.medium.com/max/2560/0*-SGR1ru7IGFnZ5iv.png)

This is not a state of the art post about the SDN and the OpenFlow
protocol. This first part contains a quick literature overview around
SDN (Software Defined Network) and the OpenFlow protocol. To skip right
into the action you can go straight to [part
2]({% link _posts/2019-07-07-Software-Defined-Network-and-the-OpenFlow-protocol-live--Part-2--698eeafe74f6.md %}).

### **Software Defined Network**

The concept of **Software Defined Network** was created in 2005, it
stands for having all networks controlled and managed by software. This
concept may be very handy to network administrators, allowing them to
dynamically manage the network, making changes without the need to
physically interconnect and organize the network.

One more refined way of defining Software Defined Network rests on the
following four pillars:

-   *Decoupling of the data and control planes*. Separating the
    **control plane** from the data plane.
-   *Forwarding decisions can be based on more information than just
    destination addresses.*
-   *Control logic/software can be moved to an external
    entity*.
-   *Network functionality is programmable via software applications
    running in conjunction with some type of network
    controller*.

The basic SDN architecture diagram is:

![SDN Architecture
diagram](https://cdn-images-1.medium.com/max/800/0*nWq1F3vRKcw_9Lha.png)

### The OpenFlow Protocol

The OpenFlow protocol allows programmers/network
administrators/researches to run networking experiments on real hardware
in real networks. The idea behind OpenFlow is to provide an abstraction
of hardware functionality that would be compatible with most of the
hardware available. A nice and quick explanation was made by Dr. G.
Bernstein:

> OpenFlow, its like an assembly language for switches.

The full [OpenFlow
Specifications](https://www.opennetworking.org/technical-communities/areas/specification) are freely available for download.

The basic OpenFlow architecture consists of a switch OpenFlow, a
controller, and a secure channel between them.

![](https://cdn-images-1.medium.com/max/800/0*9tnoemmE0Mgq0ygk.png)

### OpenFlow Switch

An OpenFlow Switch consists of at least three parts:

-   A **Flow Table**, with an action associated with each flow entry,
    to tell the switch how to process the flow.
-   A **Secure Channel** that connects the switch to a remote control
    process (called the controller), allowing commands and packets to be
    sent between a controller and the switch using
-   The **OpenFlow Protocol**, which provides an open and standard way
    for a controller to communicate with a switch.

By using the OpenFlow Protocol an **OpenFlow Switch** becomes a **dumb
datapath** element that **forwards packets between ports**, as **defined
by a** remote **control** process.

### Controllers

A controller adds and removes flow-entries from the **Flow Table** on
behalf of experiments. Isolating the control plan in a logically
centralized application, a programmer can deal with typical problems of
software development and distributed systems like:

-   Fault Tolerance
-   Complex/volatile business logic
-   Testing
-   Efficiency
-   Design

To create a controller that centralizes the application logic we will
use the [Ryu](https://osrg.github.io/ryu/index.html) library. Ryu is a component-based software-defined
networking framework, it provides a well-defined API to, among several
other things, create network controllers.

On [part 2]({% link _posts/2019-07-07-Software-Defined-Network-and-the-OpenFlow-protocol-live--Part-2--698eeafe74f6.md %}), we will see a default implementation of a switching
hub to understand how to build a simple SDN.

### References

-   [Software Defined Network
    (SDN)](https://www.grotto-networking.com/BBSDNOverview.html#sdn-overview)
-   [OpenFlow: Enabling Innovation in Campus
    Networks](http://ccr.sigcomm.org/online/files/p69-v38n2n-mckeown.pdf)
-   [O PROTOCOLO OPENFLOW
    (Pt-Br)](https://blog.pantuza.com/artigos/o-protocolo-openflow)
-   [OpenFlow
    Specifications](https://www.opennetworking.org/technical-communities/areas/specification)

---
layout: post
title: The architecture of open source applications [Asterisk]
categories: [software, open source]
excerpt: This post summarizes Chapter 1 of the book The architecture of open source applications - Vol I
last_modified_at: 2022-12-20
---

<br>

_This post summarizes Chapter 1 of the book The architecture of open source applications - Vol I_


Asterisk is a project designed to manage communication systems, its general usage is for telephony, but phones can be replaced by any other type of communication channels. An initial look left the impression of a very traditional project, on a very traditional area (telephony), using a very traditional language (C language).

A summary of the modeling of the system can be resumed in: Asterisk core interacts with many modules to enable communication of different media types between channels. Modules called channel drivers provide channels that use dialplans to execute pre-defined behavior, channels also use bridging to interact with other channels in performatic ways.

Points that caught my attention:
- Channels are a central point of the system, and they are hardware specific. The design decision to deal with ambiguity was simple: create individual classes for each hardware and make them converge to a common abstraction, allowing features to work independently of the telephony in use. This is specified by ast_channel and ast_channel_tech
- Unique language to configure dialplans. Language created is simple, expressive enough, and looks/is treated like a configuration
- Application is heavily multithreaded, to make the debug process easier, all code that uses threads passes through debug wrappers.
- Complex logic lies on codecs implementation (encode/decode media stream) and bridges implementation (optimized media transfer between channels)

I found the [project’s github](https://github.com/asterisk/asterisk) well organized and maintained, however, my lack of expertise with project pattern of C projects made it difficult to navigate through the code.

Check below a summary of my notes:

<img
 src="/images/2022-08-06-architecture-open-source-asterisk/asterisk.png" alt="Mind map of asterisk"
 title="Mind map of asterisk"
/>

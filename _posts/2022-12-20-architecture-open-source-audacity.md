---
layout: post
title: The architecture of open source applications [Audacity]
categories: [software, open source]
excerpt: This post summarizes Chapter 2 of the book The architecture of open source applications - Vol I
last_modified_at:
---

<br>

_This post summarizes Chapter 2 of the book The architecture of open source applications - Vol I_

<img
 src="/images/2022-12-20-architecture-open-source-audacity/audacity.png" alt="Audacity's logo"
 title="Audacity's logo"
/>


Audacity is a sound recorder and audio editor initially written by Dominic Mazzoni in 1999 as a platform to develop and debug audio processing algorithms.
- Open source: <https://github.com/audacity/audacity>
- Applies the principle of discoverable interface
- Code principles are "try and be consistent"
- Depends heavily on wxWidgets and PortAudio


Description of audacity codebase:
> Audacity codebase is a mix of well-structured and less well-structured code... there are some impressive buildings, but you will also find run-down neighborhoods that are more like a shanty town.


## Points that caught my attention:
- Modular design was selected to drive the project towards better hiding of internal structure. By thinking carefully of an API, the interface gets more attention and forces the team to think of better abstractions, while also helping to avoid forks

- Audacity has custom implementations for general functionalities because they were not provided initially by the libraries used

- By using wxWidgets they ran into a lot of constraints during the life of the project

- Evolutionary features are a good setup work for major changes

- PortAudio and wxWdigets come with the price that there's no flexibility to choose the abstractions. The result was less than pretty code for playback and recording because they need to handle threading in three different ways. The code also does more copying of data than it could do if they controlled the abstractions.

- The  wxWidgets API tempted the team into writing some verbose, hard-to-follow application code. The solution was to add a facade in front of wxWidgets to give better abstractions and cleaner application code.

- Existing widgets were disconsidered on the TrackPanel due to lack of support and performance. This resulted in a messy module for a while.

- A decision about what not to include in a program can be as important. It can lead to cleaner, safer code.

- Audacity is a community effort.


## Audacity polemic
- Audacity used to prioritize keeping the community happy and discourage forks, this changed in 2021

- An update on the terms of service caused a split in the community: "Data necessary for law enforcement, litigation and authorities' requests (if any)" on the grounds of "Legitimate interest of WSM Group to defend its legal rights and interests"

- By doing so, the team added networking features to the library to collect user's personal information

- This topic was scrutinized by the community and many decided to fork the libraries while audacity
  clarified privacy police


Check below a summary of my notes:

<img
 src="/images/2022-12-20-architecture-open-source-audacity/all.png" alt="Mind map of audacity"
 title="Mind map of audacity"
/>

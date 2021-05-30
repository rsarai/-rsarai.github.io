---
layout: post
title: The Memex Service API
categories: [memex]
excerpt: A write-up on how I imagine my second brain and early proposals on how to build it.
last_modified_at:
---
<br>
-   **previous:** [The Memex Building Blocks]({% link _posts/2021-05-18-the-memex-building-blocks.md %})
-   **next:**
<br><br>

1.  [Principles](#org574f1ea)
2.  [Requirements](#org3805cb7)
3.  [Anti-Goals](#orgc33c7a6)
4.  [Proposals](#org7799149)
    1.  [Proposal #0](#org439da3a)
    2.  [Proposal #1](#org5e9f6d9)
5.  [Conclusion](#org37b3c78)

<br><br>
As described in the previous post the memex has three building blocks: Exporters, Importers, and the Service API. Not much can be said on the exporters and importers, as developers we can get into implementation details and strategies, but in general, these two are straightforward. My mind only works properly when tackling the biggest problem first, therefore since the service API concentrates all the logic of the memex and it's where I will start.


<a id="org574f1ea"></a>

## Principles

The Service API should be an interface to process all my personal data for as long as I live. In addition to the idea of longevity, other principles permeate this project:

-   Disk is cheap;
-   Redundancy is good;
-   Raw data should exist and be available locally;
-   Data should never go to the cloud;
-   Event sourcing is good.

Along with the focus on personal usage, this project is designed as a learning project. This means that I'm not trying to build a commercial product or thinking about usability for multiple users. Expect to see multiple attempts to implement the same thing, expect random experiments, expect a mildly organized mess.

I will appreciate any feedback regarding the experiments, if you find something that looks very weird feel free to ping me 😊.


<a id="org3805cb7"></a>

## Requirements

-   **Functionality**: The system should be able to store all my data forever, never lose anything and allow me to search, browse, and visualize the data in multiple formats.

-   **Modifiability**: The optimal architecture is required to be mutable enough to support the addition of data or the removal of data. Even more, intrinsic components of the project should be able to be replaced by higher-capacity replacements, therefore hard-coding resource assumptions or limits should be avoided.

-   **Performance**: Data to be used in this project will grow exponentially, all the interactions between the user and the app should be fast and time-sensitive.

Besides the requirements, other ideas will be considered on the system:

-   Event-driven architecture. Logs of events should be added in an append-only manner. Objects are initially immutable, and mutable objects are represented by storing immutable, timestamped, GPG-signed blobs representing a mutation request.
-   I like the idea of having optional links. The system would not have them by default, however, I (the user) would be able to create links for things that I see as connected. The problem here would be how to make this operational since the idea behind the system is to have the data always saved on disk and have the database as a proxy.
-   To have all the data available, use the exporters to create local mirrors of external data.


<a id="orgc33c7a6"></a>

## Anti-Goals

-   Propose an online system connected to the cloud;
-   Propose an immutable specification;
-   Propose a general solution that solves all future scaling problems;
-   Propose a system designed to assure the security of the data.


<a id="org7799149"></a>

## Proposals

In general, the service API is a backend powered by some kind of database with two main endpoints: one for reading and one for writing. Proposals mostly differ on the database type, but all rely on the following assumptions:

-   Frontend of the application will be separated;
-   Database should be filled retroactively with data collected from backups;
-   Backups should exist locally as files.


<a id="org439da3a"></a>

### Proposal #0

The service API should be powered by a Postgres database with minimal tables. Four tables are initially planned, three of them have a more rigid structure and one flexible. Tables are:

-   Provider: saves all the data sources
-   Activity: saves general activities names
-   Entity: saves entities involved in activities
-   EventLog: saves raw information about the event

Note that Provider, Activity, and Entity are not required tables, they are useful for organization purposes, but their attributes should be minimal. EventLog is the schema that deals with all the memex information based on the event sourcing methodology where events are the source of truth, everything is saved as a log. The table contains connections to Provider, Activity, and Entity (as the main entity and secondary entities). Details of the event are concentrated in a JSON field which can be easily queried and filtered. Performance limitations are imminent from this approach and I'm eager to test them.

The JSON field tries to overcome the future changes in the structure of the data avoiding backward migrations and delegating the responsibility to whoever is fetching the data (the API). Changes on older models (adding more data to records) will be dealt with by creating a new timestamped EventLog that will deprecate the old EventLog.

The rigid schema definition required by relational databases makes changes on the data structure hard to evolve and we all know how migrations are time-consuming and stressful.  There are many write-ups on how managing relational databases could be more of a curse than a blessing<sup><a id="fnr.1" class="footref" href="#fn.1">1</a></sup>, however, I still see it as an approach worth considering.

As a resume, the database is a simple general schema that is not intended to change. The API is the one responsible for fetching the data and porting it to an appropriated shape, this is based on the Solid platform where decentralized data apps are allowed to interact by shaping protocols on how to deal with linked data<sup><a id="fnr.2" class="footref" href="#fn.2">2</a></sup>. Logs are immutable, therefore changes to past data should be made by creating new ones and linking/deprecating to old ones.

I can see this being a practical solution as the application reaches a few million rows. However, after this threshold I already experienced performance degradation on real-life production systems, the solution at the time was to change these types of logs to tools like Elasticsearch. I consider this type of approach an overkill for a personal project and will deal with those problems as they arrive. Meaning: I have no plan for scaling other than treat the upcoming challenges with goodwill.


<a id="org5e9f6d9"></a>

### Proposal #1

Stepping out of my comfort zone, the second proposal is experimental and vague. The service API should be powered by a document-based database such as MongoDB. Flexibility lies in the fluid-structure of the documents and minimal protection structure against duplications. This method pressures the import process (filling the database with data from the backups) and may require a better organization/automatization of the exporters' structure<sup><a id="fnr.3" class="footref" href="#fn.3">3</a></sup>.


<a id="org37b3c78"></a>

## Conclusion

One must crawl before run, this document describes the initial steps needed to build a functional second brain. I selected two database engines (MongoDB and Postgres) to experiment with and defined an initial scope to implement and test. Time to get to work 👩‍💻.

---


<br>
-   **previous:** [The Memex Building Blocks]({% link _posts/2021-05-18-the-memex-building-blocks.md %})
-   **next:**

<br>

## Footnotes

<sup><a id="fn.1" href="#fnr.1">1</a></sup> <a target="blank" href="https://beepb00p.xyz/unnecessary-db.html">Against unnecessary databases</a>

<sup><a id="fn.2" href="#fnr.2">2</a></sup> <a target="blank" href="https://ruben.verborgh.org/blog/2019/06/17/shaping-linked-data-apps">Shaping Linked Data apps</a>

<sup><a id="fn.3" href="#fnr.3">3</a></sup> The previous post on this path discusses briefly how the structure of the exporters is.

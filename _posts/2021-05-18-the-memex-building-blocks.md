---
layout: post
title: The Memex Building Blocks
categories: [memex]
excerpt: Formalizing to myself the building blocks of the memex.
last_modified_at: 2021-05-29
---
<br>
-   **previous:** [Building myself a liability]({% link _posts/2021-05-10-building-a-liability.md %})
-   **next:** [The Memex Service API]({% link _posts/2021-05-29-the-memex-service-api.md %})

<br><br>
I'm building myself a place to store, query, and analyze all my personal data extracted from the multitude of devices available. A series of posts on this blog details the how.

Many names describe this project, timeline, lifelogging, and memex. I'm very fond of the memex term since it doesn't connect to any type of commercial software or development term currently used. The memex was a device, proposed a long time ago as a way to manage personal information, details can be found [here](<https://hyfen.net/memex/>). Inspired by the link before and the work of many other open-source contributors, I'm building myself one.

Let's start from the top, this project circles around analyzing and organizing available data, including how to make different formats available and useful. Before engaging in architectural steps and design plans, the wise thing to do would be to define an initial scope.

When talking about tracking, one can always go crazy and select more than one can tackle. The multitude of categories available can be overwhelming, vide Sleep, Well-being, Fitness, Work, Media, Drink, Food, Social, Travel, Screen Time, etc. Collect and process all this data at once to make it available to the memex system right from the beginning would be extremely counter-productive. Simple is better than complex, therefore for the first version of the system I chose to contemplate four data sources:

-   Mood tracking
-   Habit tracking
-   Bash commands
-   Browser history

My mood tracking is a daily tracking with multiple options, the idea is to have a data source with very few empty days. My habit tracking system is an app that presents me with a daily binary option, the intuition here is to have something where it is possible to have a lot of days off. Bash commands and Browser history were selected for the high amount of daily items, which will allow me to test the behavior of the application as it scales. Another interesting point is that there is some similarity between the data, but not a complete overlap, this will fit for testing different schemas and no schemas at all.

**mood**
-   datetime
-   mood
-   activities
-   note

**bash**
-   datetime
-   command
-   folder
-   device

**habit**
-   name
-   description
-   datetime

**browser history**
-   datetime
-   link
-   device
-   visit count

With the data in hand, the first step should be to define a default architecture for the application. The building blocks of the Memex: Exporters, Importers, Service API.


<a id="org477d251"></a>

### Exporters

Exporters are the essential part of the system. They are responsible for downloading the data from the appropriate provider and storing it safely locally. The goal behind this project is to own my personal data, removing it from silos that I don't control, so keeping it local is a functional requirement of my project. Note, that even though I foresee the use of a database to power the memex, the data is still required to be available locally to assure consistency, ownership, and to account for future changes on the database modeling due to unexpected reasons. Exporters are also responsible for the processing of the data to a friendly format to be used by the next scripts.


<a id="org73009f6"></a>

### Importers

Have the responsibility to feed the memex with the most recent data. I will discuss implementation strategies in the next posts.


<a id="org25f2654"></a>

### Service API

Has the responsibility to deal with all the interactions of the memex, requirements include fetching the data and querying data. On top of this API, a minor frontend will provide an easy way to interact with the data. I will discuss implementation strategies in the next post.


<a id="org12a53d3"></a>

## Exporters Details

Nothing major here, exporters rely on the presence of local files on specific folders defined by a configuration config. Folders are periodically updated by cron jobs that or copy local files to correct directory<sup class="footnotes"><a id="fnr.1" class="footref" href="#fn.1">1</a></sup>, or download files from a specific drive folder, or just fetch new information from an API. Additionally, exporters read the appropriate files and process items accordingly. My default implementation is simple and can be found <a target="blank" href="https://github.com/rsarai/hq/tree/main/hq/modules">here</a>, note that I require a <a target="blank" href="https://github.com/rsarai/hq/blob/main/hq/modules/bash.py#L9">config file</a> which is not detailed inside the project but is just a python file with different classes, one for each provider.

---
<br>
Inspiration for exporters is based on the <a target="blank" href="https://github.com/karlicoss/HPI">HPI</a> package, been using for a while now and have nothing to complain about 👍🏾.

<br>
-   **previous:** [Building myself a liability]({% link _posts/2021-05-10-building-a-liability.md %})
-   **next:** [The Memex Service API]({% link _posts/2021-05-29-the-memex-service-api.md %})

<br>

## Footnotes

<sup><a id="fn.1" href="#fnr.1">1</a></sup> As is the case with bash commands which are appended to a single file and with browser history which is a sqlite file.



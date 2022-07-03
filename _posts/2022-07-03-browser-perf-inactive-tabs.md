---
layout: post
title: Browsers degraded performance in background tabs
categories: [software]
excerpt: This post describes how I discovered that the load time for interactions in background tabs is 22x to 56x slower
last_modified_at:
---

In this post:
1.  [TLDR](#org638f63e)
2.  [Context](#orgd834b73)
3.  [Technical details](#orgf9e1b7f)
4.  [Sources](#orgfdfe677)

<br>

<a id="org638f63e"></a>

## TLDR

The load time for interactions in background tabs is 22x to 56x slower. This is undoubtedly tracked by Real user monitoring (RUM) solutions but is not discriminated against when defining alerts and budgets.


<a id="orgd834b73"></a>

## Context

In a recent project, my challenge was to use available RUM (real user monitoring) metrics to define performance SLOs and error budgets. My initial thought was that the process would be simple since the historical data was available.

One special detail of the data was that it had a long tail. Imagine a distribution with an acceptable mean, a worrying p75, and a completely out-of-the-scope p90.

<img
 src="/images/2022-07-03-browser-perf-inactive-tabs/distribution.png" alt="Two distributions. One ideal and one with a long tail"
 title="Two distributions. One ideal and one with a long tail"
/>

Outliers could not be excluded by analyzing percentiles, resulting in completely obscene performance metrics. This got me into an spiral of research and fear until I decided to analyze and sample the slow pages myself, to my surprise results were always close to the p50/p75.

One thing we noticed was that most of the slow events had “in foreground periods”, which was how our provider signaled moments where the user had put the page in the background and returned to it. This appeared to be a common thing among users, the realization paved the way to the discovery that some browsers (chrome in particular) slow down (or even suspend) execution of JavaScript on unfocused tabs.


<a id="orgf9e1b7f"></a>

## Technical details

Extracted from different posts:

-   Background tabs set a per-frame simultaneous loading limit lower than for the foreground tabs. For example, Google Chrome limits the number of resources fetched to six when the tab is in the focus and to three when in the background per server/proxy. What it means is that only a limited number of requests from the browser are permitted to go to the network stack in parallel.
-   Background tabs are also restricted to CPU access. Google Chrome limits timers in the background to run only once per second. In addition, Chrome will delay timers to limit average CPU load to 1% of the processor core when running in background.
-   Load time difference when running in background
    -   median load time for background interactions was 24x worse,
    -   90th percentile was 22 times slower,
    -   99th percentile loaded 56 times slower


<a id="orgfdfe677"></a>

# Sources

-   <https://dzone.com/articles/did-you-know-that-background-tabs-in-your-browser>
-   <https://chromestatus.com/feature/5527160148197376>
-   <https://stackoverflow.com/questions/6032429/chrome-timeouts-interval-suspended-in-background-tabs>
-   <https://plumbr.io/blog/performance-blog/background-tabs-in-browser-load-20-times-slower>
-   <https://newrelic.com/blog/best-practices/expected-distributions-website-response-times>


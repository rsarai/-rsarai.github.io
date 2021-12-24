---
layout: post
title: Creating a web scraper out of nowhere part 1
categories: [code]
excerpt: ""
---

_Originally published on medium_

![](https://cdn-images-1.medium.com/max/2560/1*H2KbxexBfq9MhjU2BcLuYA.jpeg)

As an almost computer engineer it's little annoying being called "the
computer girl" or the person that you think that knows how to fix the TV
signal (although my course is computer engineering, not
telecommunications engineering). Jokes (or hints) aside, sometimes some
friends bring some interesting ideas which become great challenges for
my time off.

The problem of this week was: Someone had two sites the older had the
content, but as a wise person once say "new is always better". The only
problem with the new one was that it lacked content.

![The
rule?](https://cdn-images-1.medium.com/max/800/1*SmjqCRtukpE74N47dJ9DZA.gif)

![Hell
yes!](https://cdn-images-1.medium.com/max/800/1*hZt2Y4a3J52hWE-u5G7FYw.gif)

As expected the owner didn't have access to the old database and the old
website would be deactivated soon.

The job was to get all the information on the website (text, images,
links, everything) and prepare this content for the new site. My first
thought was: "What was the name of the thing, which scans websites and
gets the content? Crawler... scraper... scrawler... ?"

> With the help of some friends on Quora, I found out that:\
> \
> **Crawling** refers to automatically retrieving web pages and
> following links to find still more web pages.

> **Scraping** means parsing those pages to extract pieces of
> information in a structured way. It also refers to creating a
> programmatic interface, an API, that interacts with a site through an
> HTML interface meant for humans.

As a matter of effect, this subtle difference doesn't matter for the
application, since both concepts were used to solve this problem.

For implementation, I used Scrapy that is an application framework
written in Python for crawling web sites and extracting structured data.
Even though Scrapy was originally designed for web scraping, it can also
be used to extract data using APIs (such as Amazon Associates Web
Services) or as a general-purpose web crawler.

If you want to do a simple thing and don't wanna dive into a new lib,
[just use a Python lib called
BeautifulSoup](https://gist.github.com/rsarai/b8a7db3289827b5f1f2e43cf6502c7a8)

### THE PROCESS 1. Install Scrapy

As a first step install Scrapy. Is very recommended that you install
Scrapy in a dedicated virtualenv, to avoid conflicting with your system
packages.

```bash
pip install Scrapy
```

### 2. Create a project

```bash
scrapy startproject newproject
```

This will create a newproject directory with all that you need.

### 3. Exporting the scraped data using the command line

In my job, I use Django the web framework for perfectionists with
deadlines. One of my favorites features is the shell. There I can
execute python scripts for understand the behavior of the code. And
scrapy has a similar feature. The Scrapy shell is an interactive shell
where you can try and debug your scraping code very quickly, without
having to run the spider. One The best way to learn how to extract data
with Scrapy is trying selectors using the shell.

```bash
scrapy shell 'url'
```

The shell is used for testing XPath or CSS expressions to see how they
work and what data they extract from the web pages you're trying to
scrape. Here is a example:

```bash
>>> body = '<html><body><span>good</span></body></html>'
>>> Selector(text=body).xpath('//span/text()').extract()
[u'good']
```

For more check the [Scrapy
documentation](https://doc.scrapy.org/en/latest/topics/selectors.html).

-----

Spiders are classes that you define and that Scrapy uses to scrape
information from a website. They must subclass
`scrapy.Spider` and define the initial
requests to make. In my case, originally, I wanted to navigate through
all the product pages.

Go inside the newproject \> spider folder and create a new file called
\`products_spider.py\` or any other name you want:

What exactly this code is doing? Well, in **ProductsSpider** at the
beginning you a setting a unique name to represent your Spider. This is
very important, because you will use this name to run your spider later.

After, you are in the start_request method generating
`scrapy.Request` objects from URLs or you
can just define a `start_urls` class
attribute with a list of URLs. This list will then be used by the
default implementation of `start_requests().`{.markup--code
.markup--p-code .u-paddingRight0 .u-marginRight0}

For extracting the data aScrapy spider typically generates many
dictionaries containing the data extracted from the page. To do that, we
use the `yield` Python keyword in the
callback.

Running your spider must be simple, you can do it typing:

```bash
scrapy crawl products
```

The result of this will be something like:

```
... (omitted for brevity)
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Spider opened
2016-12-16 21:24:05 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2016-12-16 21:24:05 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://supersecretsite.com/robots.txt> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://supersecretsite.com/page/2/> (referer: None)
2016-12-16 21:24:05 [scrapy.core.engine] INFO: Closing spider (finished)
...
```

### 5. Export the data to your own spreadsheet

For exporting the data you just collect you just have to set this in
your configurations file: `FEED_FORMAT`
'csv'. And run this command:

```bash
scrapy crawl products -o spreadsheetname.csv
```

Wow! This is getting big, let's save some room for part 2.

In part 2 we will talk about how to download multiple images
(**ProductImgSpider**) and how to rename them.

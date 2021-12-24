---
layout: post
title: Creating a web scraper out of nowhere Part.2
categories: [code]
excerpt: ""
---

_Originally published on medium_

![Please write the part 2. Please write the
part 2.](https://cdn-images-1.medium.com/max/800/1*lTwQrvqPeT2Gk7TqvhvFFg.gif)

Attending the many requests I receive this week (wrong, it was only
one), I brought the second part of creating a web scraper out of
nowhere. If you don't read the first part, you check
[here](https://medium.com/the-computer-engineer-weekly-code-challenge/creating-a-web-scraper-out-of-nowhere-part-1-830c2bb272c0).

As I say in part 1, the job was to retrieve data from a website. One big
part of this data where images, totalizing almost 600 items, each image
was on your own page and should be downloaded with the name of the page.

For this, we use **Scrapy** a Python library. With scrapy, you can
easily download images from websites with the **ImagesPipeline**.

### Download Pillow

Pillow is imaging library that scrapy uses it for thumbnailing and
normalizing images to JPEG/RGB format.

```bash
pip install Pillow
```

### Enable ImagesPipeline*.*

In your *settings.py* include:

```python
ITEM_PIPELINES = {'scrapy.pipelines.images.ImagesPipeline': 1}
```

and configure the target storage setting to a valid value that will be
used for storing the downloaded images. Otherwise the pipeline will
remain disabled, even if you include it in the
`ITEM_PIPELINES` setting.

```python
IMAGES_STORE = '/path/to/valid/dir'
```

### Create a item

In your main directory, if not exists create a file **items.py.**

```python
class ProductsItem(scrapy.Item):
    image_urls = scrapy.Field()
    images = scrapy.Field()
    #other fields...
```

### Scrape an item and add URLs.

In your Spider, scrape an item and put the URLs of the desired into a
**image_urls** field (*must be a list*).

The item is returned from the spider and goes to the ImagePipeline. The
URLs in the `image_urls` field are
scheduled for download using the standard Scrapy scheduler and
downloader (which means the scheduler and downloader middlewares are
reused), but with a higher priority, processing them before other pages
are scraped. The item remains "locked" at that particular pipeline stage
until the files have finish downloading (or fail for some reason).

### Create custom filenames

The files are stored using a [SHA1
hash](https://en.wikipedia.org/wiki/SHA_hash_functions) of their URLs for the file names. e.g.:

-   For this URL:
    *http:****//****www****.****example****.****com****/****image****.****jpg*
-   The filename would be:
    *3afec3b4765f8f0a07b78f98c07b83f013567a0a*

As cool as I think that SHA1 hash is, this nomenclature couldn't be used
in a real system.

For overriding the images name you have create a new Pipeline and extend
the *ImagesPipeline.*

To finish don't forget to update the `ITEM_PIPELINES` to use the new file that we just write.

```python
ITEM_PIPELINES = {'scrapy.pipelines.images.ScrapymercadoPipeline': 1}
```

And that's all. See you soon and buy bitcoin!

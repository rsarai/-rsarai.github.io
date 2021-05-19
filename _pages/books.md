---
layout: default
permalink: /books/
title: Literatura
---

<div class="page"><h1>Literatura</h1></div>
<br>
<div class="posts">
    {% for category in site.categories %}
        {% capture category_name %}{{ category | first }}{% endcapture %}
        {% if category_name == 'books' %}
            {% for post in site.categories[category_name] %}
                <article class="post">
                    <a href="{{ site.baseurl }}{{ post.url }}">
                        <h2>{{ post.title }}</h2>
                        <div>
                            <p class="post_date">{{ post.date | date: "%B %e, %Y" }}</p>
                        </div>
                    </a>
                    <a href="{{ site.baseurl }}{{ post.url }}" class="read-more">Read More</a>
                </article>
            {% endfor %}
        {% endif %}
    {% endfor %}
</div>

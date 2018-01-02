---
layout: page
title: Welcome My World!
tagline: Supporting tagline
---
{% include JB/setup %}

Hi!
Share open project with you.
Here is my list..
[github.com/edydkim](https://github.com/edydkim/)

## Update Author Attributes

See more below:

[LinkedIn](https://www.linkedin.com/in/edydkim/)
[app-creative.com](http://app-creative.com)

## Posts

Here's a "posts list".

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## To-Do

This site is still unfinished and updating.



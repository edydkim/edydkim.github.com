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

## Author Attributes

See more below:

[LinkedIn](https://www.linkedin.com/in/edydkim/)

[App-Creative.com](http://app-creative.com)

## Posts

Here's a "posts list".

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## Notices

For Data Science Engineering, an assignment of ETL & Preprocessing to a new joiner and its solution are created/uncovered for a fun :
[Welcome to Data Science Engineering Pages](https://edydkim.github.io/dse-interview/)

For R programming, the following solution would be useful in performance tuning as to another assignment: 
[Welcome to R Programming Assignment Solution Pages](https://edydkim.github.io/ProgrammingAssignment2/)

## To-Do

Keep updating!



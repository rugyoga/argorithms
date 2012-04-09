---
layout: page
title: Argorithms
---
{% include JB/setup %}

## About me

A self-confessed algorithms nerd.

I hack in ruby, Java, python.

When not hacking, I enjoy Pool, Car racing, Scrabble, Chess.
    
## The story so far

Here are my most recent posts:

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## Suggestions

I'm always open to suggestions about what algorithms and data structures to dig into.

Most of the posts here are the result of some practical need in one of my day jobs.


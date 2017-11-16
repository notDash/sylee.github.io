---
# You don't need to edit this file, it's empty on purpose.
# Edit theme's home layout instead if you wanna make some changes
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
layout: default
title: 李生勇的技术日志
---

<h2>{{ page.title }}</h2>

<p>Recent Blog Posts</p>

<ul>
    {% for post in site.posts %}
      <li>{{ post.date | date_to_string }}
      <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
      </li>
    {% endfor %}
</ul>
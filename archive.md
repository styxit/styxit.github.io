---
layout: page
title: Archive
description: Post archive
---

# Post archive

{% for post in site.posts %}
- {{ post.date | date: "%x" }} :: [{{ post.title }}]({{ post.url }}){% endfor %}
---
layout: default
---

Nơi đây cất giấu những bài viết thú vị về **lập trình**, **phân tích dữ liệu** và nhiều chủ đề khác.

<ul>
{% assign date_format = site.hacker.data_format | default: "%b %-d, %Y" %}
{% for post in site.posts %}
<li>
<span>{{ post.date | date: date_format }}</span>
<h3><a href="{{ post.url }}" style="color:inherit;text-decoration:none">{{ post.title | escape }}</a></h3>
{{ post.excerpt }}
</li>
{% endfor %}
</ul>

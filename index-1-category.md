---
layout: page
title:  Categories
permalink: /category/
---
{% for category in site.categories %}
<h3>{{ category | first }}({{ category | last | size }})</h3>
<ul class="arc-list">
    {% for post in category.last %}
        <li>{{ post.date | date:"%d/%m/%Y"}} <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>

{% endfor %}
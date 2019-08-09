---
layout: default
---

<p>
    <a href="{{ site.url }}"><< Return Home</a>
</p>

{% for tag in site.tags %} 
[root@wangspx.github.io ~]# ll <a id="{{ tag[0] }}" href="#{{ tag[0] }}">./{{ tag[0] }}</a>
<ul class="index">
    {% assign posts = tag[1] %}  
        <p>total: {{ posts | size }}</p>
        {% for post in posts %}
            {% include articles-list.html %}
        {% endfor %}
    {% assign posts = nil %}
</ul>
{% endfor %}


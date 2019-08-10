---
layout: default
---

[root@wangspx.github.io ~]# pwd

wangspx.github.io

[root@wangspx.github.io ~]# echo $MY_GITHUB

[https://github.com/wangspx](https://github.com/wangspx)

[root@wangspx.github.io ~]# ls <a href="{{ site.url }}/tags">./tags</a>

<div>
    <p>total: {{site.tags | size}}</p>
    <p>
    {% for tag in site.tags %}
        <a class="post-tags-item" href="{{ site.url }}/tags#{{ tag | first }}">{{ tag | first }}</a>
    {% endfor %}
    </p>
</div>

[root@wangspx.github.io ~]# ll <a href="{{ site.url }}/articles">./articles</a> <span> | head -n 5</span>

<div>
    <p>total: {{site.categories.article | size}}</p>
    {% for post in site.categories.article limit:5 %}
        {% include articles-list.html %}
    {% endfor %}
    <p><a href="{{ site.url }}/articles">more...</a></p>
</div>

[root@wangspx.github.io ~]# <i class="line" />




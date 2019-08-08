---
layout: default
---

[root@iZwz9j80yesn9eqdugq563Z ~]# pwd

wangspx.github.io

[root@iZwz9j80yesn9eqdugq563Z ~]# echo $MY_GITHUB

[https://github.com/wangspx](https://github.com/wangspx)

[root@iZwz9j80yesn9eqdugq563Z ~]# ls <a href="{{ site.url }}/articles">./tags</a>

<div>
    <p>total: {{site.tags | size}}</p>
    <p>
    {% for tag in site.tags %}
        <a class="post-tags-item" href="{{ page.url }}?keyword={{ tag | first }}">{{ tag | first }}</a>
    {% endfor %}
    </p>
</div>

[root@iZwz9j80yesn9eqdugq563Z ~]# ll <a href="{{ site.url }}/articles">./articles</a> <span> | head -n 5</span>

<div>
    <p>total: {{site.categories.article | size}}</p>
    {% for post in site.categories.article limit:5 %}
        {% include articles-list.html %}
    {% endfor %}
    <p><a href="{{ site.url }}/articles">more...</a></p>
</div>

[root@iZwz9j80yesn9eqdugq563Z ~]# ll <a href="{{ site.url }}/articles">./note</a>




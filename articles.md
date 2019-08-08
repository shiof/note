---
layout: default
---

[root@iZwz9j80yesn9eqdugq563Z ~]# pwd

wangspx.github.io/articles

[root@iZwz9j80yesn9eqdugq563Z ~]# <span>ll | head -n 5</span>

<div>
    <p>total: {{site.categories.article | size}}</p>
    {% for post in site.categories.article limit:5 %}
        <div class="row">
            <div class="col pr-0">
                <span>-rwxr-xr-x. 1 </span>
                <span>
                    {% assign tag = post.tags | sort %}
                        {% for category in tag %}
                         <span><a href="{{ site.url }}category/#{{ category }}" class="reserved">{{ category }}</a> </span>
                        {% endfor %}
                    {% assign tag = nil %}
                </span>
                <span class="float-right text-right">{{ post.date | date: "%b %d %Y" }}</span>
                <span class="px-3 float-right text-right">{{ post.content | number_of_words }}</span>
            </div>
            <div class="col-8"><a class="post-link" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a></div>
        </div>
    {% endfor %}
    <p><a href="">more...</a></p>
</div>


<!-- 遍历分页后的文章 -->
{% for post in paginator.posts %}
  <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
  <p class="author">
    <span class="date">{{ post.date }}</span>
  </p>
  <div class="content">
    {{ post.content }}
  </div>
{% endfor %}
 
<!-- 分页链接 -->
<div class="pagination">
  {% if paginator.previous_page %}
    <a href="/page{{ paginator.previous_page }}" class="previous">Previous</a>
  {% else %}
    <span class="previous">Previous</span>
  {% endif %}
  <span class="page_number ">Page: {{ paginator.page }} of {{ paginator.total_pages }}</span>
  {% if paginator.next_page %}
    <a href="/page{{ paginator.next_page }}" class="next">Next</a>
  {% else %}
    <span class="next ">Next</span>
  {% endif %}
</div>

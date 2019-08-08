---
layout: default
---

[root@iZwz9j80yesn9eqdugq563Z ~]# pwd

wangspx.github.io

[root@iZwz9j80yesn9eqdugq563Z ~]# echo $MY_GITHUB

[https://github.com/wangspx](https://github.com/wangspx)

[root@iZwz9j80yesn9eqdugq563Z ~]# ls <a href="{{ site.url }}/articles">wangspx.github.io/tags</a>

<div>
    <p>total: {{site.tags | size}}</p>
    <p>
    {% for tag in site.tags %}
        <a class="post-tags-item" href="{{ page.url }}?keyword={{ tag | first }}">{{ tag | first }}</a>
    {% endfor %}
    </p>
</div>

[root@iZwz9j80yesn9eqdugq563Z ~]# ll <a href="{{ site.url }}/articles">wangspx.github.io/articles</a> <span> | head -n 5</span>

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

[root@iZwz9j80yesn9eqdugq563Z ~]# ll <a href="{{ site.url }}/articles">wangspx.github.io/note</a>

  {% if paginator.total_pages > 1 %}
  <div class="pagination">
    {% if paginator.previous_page == 1 %}
    <a href="{{ '/' | prepend: site.baseurl | replace: '//', '/' }}" class="page-item">&laquo;</a>
    {% elsif paginator.previous_page%}
    <a href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}" class="page-item">&laquo;</a>
    {% else %}
    <span class="page-item">&laquo;</span>
    {% endif %} {% for page in (1..paginator.total_pages) %} {% if page == paginator.page %}
    <span class="page-item">{{ page }}</span>
    {% elsif page == 1 %}
    <a href="{{ '/' | prepend: site.baseurl | replace: '//', '/' }}" class="page-item">{{ page }}</a>
    {% else %}
    <a href="{{ site.paginate_path | prepend: site.baseurl | replace: '//', '/' | replace: ':num', page }}" class="page-item">{{ page }}</a>
    {% endif %} {% endfor %} {% if paginator.next_page %}
    <a href="{{ paginator.next_page_path | prepend: site.baseurl | replace: '//', '/' }}" class="page-item">&raquo;</a>
    {% else %}
    <span class="page-item">&raquo;</span>
    {% endif %}
  </div>
  {% endif %}




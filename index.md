---
layout: home
---

~~~

 __      __                                             
/\ \  __/\ \                                            
\ \ \/\ \ \ \     __      ___      __     ____  _____   
 \ \ \ \ \ \ \  /'__`\  /' _ `\  /'_ `\  /',__\/\ '__`\ 
  \ \ \_/ \_\ \/\ \L\.\_/\ \/\ \/\ \L\ \/\__, `\ \ \L\ \
   \ `\___x___/\ \__/.\_\ \_\ \_\ \____ \/\____/\ \ ,__/
    '\/__//__/  \/__/\/_/\/_/\/_/\/___L\ \/___/  \ \ \/ 
                                   /\____/        \ \_\ 
                                   \_/__/          \/_/ 
             I am just a person calling the interface...

~~~

[root@iZwz9j80yesn9eqdugq563Z ~]# ll <a href="{{ site.baseurl }}/articles">wangspx.github.io/articles</a> <span> | head -n 5</span>

<div>
    <p>total: {{site.categories.article | size}}</p>
    {% for post in site.categories.article %}
        <div class="row article-row">
            <div class="col-sm">-rwxr-xr-x. 1 </div>
            <div class="col-sm">
            {% assign tag = post.tags | sort %}
                {% for category in tag %}
                 <span><a href="{{ site.baseurl }}category/#{{ category }}" class="reserved">{{ category }}</a> </span>
                {% endfor %}
            {% assign tag = nil %}
            </div>
            <div class="col-sm text-right">{{ post.content | number_of_words }}</div>
            <div class="col-sm text-right">{{ post.date | date: "%b %d %Y" }}</div>
            <div class="col-sm-8"><a class="post-link" href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></div>
        </div>
    {% endfor %}
    <p><a href="">more</a></p>
</div>

[root@iZwz9j80yesn9eqdugq563Z ~]# ll <a href="{{ site.baseurl }}/articles">wangspx.github.io/note</a>

[root@iZwz9j80yesn9eqdugq563Z ~]# ls <a href="{{ site.baseurl }}/articles">wangspx.github.io/tags</a>

<div>
<p>total: {{site.tags | size}}</p>
{% for tag in site.tags %}
    {% assign count = tag | last | size %}
    {% assign fontsize = count | times: 4 %}
    <a class="post-tags-item" href="{{ page.url }}?keyword={{ tag | first }}">{{ tag | first }}</a>
{% endfor %}
</div>




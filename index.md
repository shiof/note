---
layout: default
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

[root@iZwz9j80yesn9eqdugq563Z ~]# echo $MY_GITHUB

[https://github.com/wangspx](https://github.com/wangspx)

[root@iZwz9j80yesn9eqdugq563Z ~]# ls <a href="{{ site.baseurl }}/articles">wangspx.github.io/tags</a>

<div>
<p>total: {{site.tags | size}}</p>
<p>
{% for tag in site.tags %}
    <a class="post-tags-item" href="{{ page.url }}?keyword={{ tag | first }}">{{ tag | first }}</a>
{% endfor %}
</p>
</div>

[root@iZwz9j80yesn9eqdugq563Z ~]# ll <a href="{{ site.baseurl }}/articles">wangspx.github.io/articles</a> <span> | head -n 5</span>

<div>
    <p>total: {{site.categories.article | size}}</p>
    {% for post in site.categories.article limit:5 %}
        <div class="row">
            <div class="col pr-0">
                <span>-rwxr-xr-x. 1 </span>
                <span>
                    {% assign tag = post.tags | sort %}
                        {% for category in tag %}
                         <span><a href="{{ site.baseurl }}category/#{{ category }}" class="reserved">{{ category }}</a> </span>
                        {% endfor %}
                    {% assign tag = nil %}
                </span>
                <span class="float-right text-right">{{ post.date | date: "%b %d %Y" }}</span>
                <span class="px-3 float-right text-right">{{ post.content | number_of_words }}</span>
            </div>
            <div class="col-8"><a class="post-link" href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></div>
        </div>
    {% endfor %}
    <p><a href="">more...</a></p>
</div>

[root@iZwz9j80yesn9eqdugq563Z ~]# ll <a href="{{ site.baseurl }}/articles">wangspx.github.io/note</a>




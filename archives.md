---
layout: default
title: "归档：Archives"
---
<ul class="list-unstyled">
     {% for post in site.posts limit:100 %} 
	 {% unless post.next %} 
    <h2>{{ post.date | date: '%Y' }}</h2> 
	{% else %} {% capture year %}{{ post.date | date: '%Y' }}{% endcapture %} {% capture nyear %}{{ post.next.date | date: '%Y' }}{% endcapture %} 
	{% if year != nyear %} 
    <h2>{{ post.date | date: '%Y' }}</h2> {% endif %} 
	{% endunless %} 
    <li><span>{{ post.date | date: "%Y-%m-%d" }}</span> <a href="{{ post.url }}">{{ post.title }}</a></li> 
	{% endfor %} 
</ul> 

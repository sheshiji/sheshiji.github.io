---
layout: default
title: "分类：Categories"
---
<ul class="list-unstyled">
{% for cat in site.categories %} 
	{% if cat[0] != 'blog' %} 
   <a name="{{ cat[0] }}"></a>
   <h2>{{ cat[0] }}({{ cat[1].size }})</h2> 
     {% for post in cat[1] %} 
    <li><span>{{ post.date | date: "%Y-%m-%d" }}</span> <a href="{{ post.url }}">{{ post.title }}</a></li>
	{% endfor %} 
   {% endif %} 
{% endfor %} 
</ul>

---
layout: page
title: Home
tagline: 
---
{% include JB/setup %}

This is Jianfei Hu's blog, recording some computer science related blogs.

Communication brings progress! Any good ideas and suggestions are welcome.
    
###Post List

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

Vistors' Map
<div id="clustrmaps-widget"></div>
<script type="text/javascript">var _clustrmaps = {'url' : 'http://bluesea147.github.com/', 'user' : 1045855, 'server' : '4', 'id' : 'clustrmaps-widget', 'version' : 1, 'date' : '2012-09-22', 'lang' : 'zh', 'corners' : 'square' };(function (){ var s = document.createElement('script'); s.type = 'text/javascript'; s.async = true; s.src = 'http://www4.clustrmaps.com/counter/map.js'; var x = document.getElementsByTagName('script')[0]; x.parentNode.insertBefore(s, x);})();</script><noscript><a href="http://www4.clustrmaps.com/user/70bff55f"><img src="http://www4.clustrmaps.com/stats/maps-no_clusters/bluesea147.github.com--thumb.jpg" alt="Locations of visitors to this page" /></a></noscript>


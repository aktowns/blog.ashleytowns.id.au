---
layout: page
title: Hello World!
---
<link rel="stylesheet" href="homepage.css" />
<div id="contain">
<div id="face" class="head">
<div id="eye">
	<div id="l">&nbsp;</div>
	<div id="r">&nbsp;</div>
</div>
<div id="eye">
	<div id="l">&nbsp;</div>
	<div id="r">&nbsp;</div>
</div>
<div id="chin"><div id="mouth">&nbsp;</div></div>
</div>
</div>
<br />
<h3>rant rant</h3>
<ul class="posts">
  {% for post in site.posts %}
	<li>
		<span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a> {{ if tags first != "" }} <span class="tags">| {{ post.tags | join ", " }}  {{ end if }}</span>
		<br />
		<span class="description">/* {{ post.description }} */</span>
	</li>
  {% endfor %}
</ul>
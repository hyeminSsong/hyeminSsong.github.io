---
layout: default
title: "Home"
---
<section style="text-align:center; padding:60px 0">
  <h1>Hi, I am {{ site.name }}</h1>
  <p>{{ site.user_title }}</p>
</section>

<h2>Recent Articles</h2>
<ul>
{% for post in site.posts limit:10 %}
  <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <small> · {{ post.date | date: "%Y-%m-%d" }}</small></li>
{% endfor %}
</ul>
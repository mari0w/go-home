---
layout: home
---

<div class="recent-posts">
<h3>📝 近期文章</h3>
<ul>
{% for post in site.posts | sort: 'date' | reverse limit: 10 %}
<li>
  <a href="{{ post.url | relative_url }}">
    <span>{{ post.title }}</span>
    <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
  </a>
</li>
{% endfor %}
</ul>
</div>

<div class="category-list">
<h3>📂 文章分类</h3>
<ul>
{% assign all_categories = site.posts | map: 'categories' | flatten | uniq | compact %}
{% for category in all_categories %}
<li>
  <a href="{{ '/categories.html' | relative_url }}#{{ category | slugify }}">
    <span>{{ category }}</span>
    <span class="badge">{{ site.posts | where_exp: "post", "post.categories contains category" | size }}</span>
  </a>
</li>
{% endfor %}
</ul>
</div>




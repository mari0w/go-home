---
layout: default
---

<div class="site-header">
  <div class="header-content">
    <a id="a-title" href="{{ '/' | relative_url }}">
      <h1>{{ site.title | default: site.github.repository_name }}</h1>
    </a>
    <h2>{{ site.description | default: site.github.project_tagline }}</h2>
  </div>
</div>

<div class="main-container">
  <div class="posts-section">
    {% for post in site.posts limit: 20 %}
    <article class="post-preview">
      <div class="post-content">
        <h3 class="post-title">
          <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        </h3>
        
        <div class="post-meta">
          <span class="post-date">{{ post.date | date: "%Y年%m月%d日" }}</span>
          {% if post.categories %}
          <span class="post-category">
            {% for category in post.categories %}
              <span class="category-tag">{{ category }}</span>
            {% endfor %}
          </span>
          {% endif %}
        </div>
        
        <div class="post-excerpt">
          {% if post.excerpt %}
            {{ post.excerpt | strip_html | truncate: 200 }}
          {% else %}
            {{ post.content | strip_html | truncate: 200 }}
          {% endif %}
        </div>
        
        <a href="{{ post.url | relative_url }}" class="read-more">阅读全文 →</a>
      </div>
    </article>
    {% endfor %}
  </div>

  <aside class="sidebar">
    <div class="widget">
      <h3 class="widget-title">文章分类</h3>
      <ul class="category-list">
        {% assign all_categories = site.posts | map: 'categories' | flatten | uniq | compact | sort %}
        {% for category in all_categories %}
        <li>
          <a href="{{ '/categories.html' | relative_url }}#{{ category | slugify }}">
            <span>{{ category }}</span>
            <span class="count">{{ site.posts | where_exp: "post", "post.categories contains category" | size }}</span>
          </a>
        </li>
        {% endfor %}
      </ul>
    </div>
    
    <div class="widget">
      <h3 class="widget-title">标签云</h3>
      <div class="tag-cloud">
        {% assign all_tags = site.posts | map: 'tags' | flatten | uniq | compact | sort %}
        {% for tag in all_tags %}
          <a href="#" class="tag-item">{{ tag }}</a>
        {% endfor %}
      </div>
    </div>
  </aside>
</div>


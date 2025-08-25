---
layout: home
---

<div class="main-container">
  <div class="posts-section">
    <h2 class="section-title">最新文章</h2>
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

<style>
.main-container {
  width: 100%;
  margin: 10px 0 0 0;
  padding: 0;
  display: flex;
  gap: 50px;
}

.posts-section {
  flex: 1;
  min-width: 0;
}

.section-title {
  font-size: 1.8em;
  margin: 0 0 30px 0;
  padding-bottom: 10px;
  border-bottom: 2px solid #333;
}

.post-preview {
  margin-bottom: 35px;
  padding-bottom: 35px;
  border-bottom: 1px solid #e0e0e0;
}

.post-preview:last-child {
  border-bottom: none;
}

.post-title {
  margin: 0 0 12px 0;
  font-size: 1.6em;
  line-height: 1.3;
}

.post-title a {
  color: #333;
  text-decoration: none;
  transition: color 0.3s;
}

.post-title a:hover {
  color: #0066cc;
}

.post-meta {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 15px;
  font-size: 0.9em;
  color: #666;
}

.post-date {
  color: #999;
}

.post-category {
  display: flex;
  gap: 8px;
}

.category-tag {
  background-color: #f0f0f0;
  padding: 4px 12px;
  border-radius: 3px;
  font-size: 0.85em;
  color: #555;
}

.post-excerpt {
  line-height: 1.7;
  color: #555;
  margin-bottom: 15px;
  font-size: 1em;
}

.read-more {
  color: #0066cc;
  text-decoration: none;
  font-size: 0.95em;
  font-weight: 500;
}

.read-more:hover {
  text-decoration: underline;
}

/* 侧边栏 */
.sidebar {
  width: 280px;
  flex-shrink: 0;
  padding-right: 0;
}

.widget {
  background: #f8f8f8;
  padding: 20px;
  margin-bottom: 30px;
  border-radius: 5px;
}

.widget-title {
  font-size: 1.2em;
  margin: 0 0 15px 0;
  padding-bottom: 10px;
  border-bottom: 2px solid #ddd;
}

.category-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.category-list li {
  margin-bottom: 10px;
}

.category-list a {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #555;
  text-decoration: none;
  padding: 5px 0;
  transition: color 0.3s;
}

.category-list a:hover {
  color: #0066cc;
}

.category-list .count {
  background: #e0e0e0;
  padding: 2px 8px;
  border-radius: 10px;
  font-size: 0.85em;
  color: #666;
}

.tag-cloud {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.tag-item {
  background: #fff;
  padding: 5px 12px;
  border-radius: 15px;
  font-size: 0.9em;
  color: #666;
  text-decoration: none;
  border: 1px solid #ddd;
  transition: all 0.3s;
}

.tag-item:hover {
  background: #0066cc;
  color: #fff;
  border-color: #0066cc;
}

/* 响应式设计 */
@media (max-width: 968px) {
  .main-container {
    flex-direction: column;
    gap: 30px;
  }
  
  .sidebar {
    width: 100%;
  }
  
  .widget {
    padding: 15px;
  }
}

@media (max-width: 768px) {
  .main-container {
    padding: 0 15px;
  }
  
  .post-title {
    font-size: 1.4em;
  }
  
  .post-meta {
    flex-wrap: wrap;
    font-size: 0.85em;
  }
}
</style>
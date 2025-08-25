---
layout: home
---

<div class="posts-container">
  {% for post in site.posts limit: 20 %}
  <article class="post-preview">
    <div class="post-content">
      <h2 class="post-title">
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      </h2>
      
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
          {{ post.excerpt | strip_html | truncate: 150 }}
        {% else %}
          {{ post.content | strip_html | truncate: 150 }}
        {% endif %}
      </div>
      
      <a href="{{ post.url | relative_url }}" class="read-more">阅读全文 →</a>
    </div>
  </article>
  {% endfor %}
</div>

<style>
.posts-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.post-preview {
  margin-bottom: 40px;
  padding-bottom: 30px;
  border-bottom: 1px solid #e0e0e0;
}

.post-preview:last-child {
  border-bottom: none;
}

.post-title {
  margin: 0 0 10px 0;
  font-size: 1.5em;
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
  padding: 3px 10px;
  border-radius: 3px;
  font-size: 0.85em;
  color: #555;
}

.post-excerpt {
  line-height: 1.6;
  color: #555;
  margin-bottom: 15px;
  font-size: 0.95em;
}

.read-more {
  color: #0066cc;
  text-decoration: none;
  font-size: 0.9em;
  font-weight: 500;
}

.read-more:hover {
  text-decoration: underline;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .posts-container {
    padding: 15px;
  }
  
  .post-title {
    font-size: 1.3em;
  }
  
  .post-meta {
    flex-wrap: wrap;
    font-size: 0.85em;
  }
}
</style>
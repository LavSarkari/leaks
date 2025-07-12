---
layout: default
title: "Leaks"
permalink: /leaks/
icon: fas fa-newspaper
order: 1
---

<h1>All Blog Posts</h1>

{% include lang.html %}

<!-- 🔽 Filter by Category -->
<div class="mb-4">
  <label for="category-filter" class="form-label fw-bold">Filter by Category:</label>
  <select id="category-filter" class="form-select" onchange="filterByCategory(this.value)">
    <option value="all">All</option>
    {% assign all_categories = "" | split: "" %}
    {% for post in site.posts %}
      {% for category in post.categories %}
        {% unless all_categories contains category %}
          {% assign all_categories = all_categories | push: category %}
        {% endunless %}
      {% endfor %}
    {% endfor %}
    {% assign sorted_categories = all_categories | sort %}
    {% for category in sorted_categories %}
      <option value="{{ category }}">{{ category }}</option>
    {% endfor %}
  </select>
</div>

<!-- 🧠 Split pinned and unpinned -->
{% assign all_pinned = '' | split: '' %}
{% assign all_normal = '' | split: '' %}

{% for post in site.posts %}
  {% if post.pin %}
    {% assign all_pinned = all_pinned | push: post %}
  {% elsif post.hidden != true %}
    {% assign all_normal = all_normal | push: post %}
  {% endif %}
{% endfor %}

{% assign all_normal = all_normal | sort: "date" | reverse %}
{% assign posts = all_pinned | concat: all_normal %}

<div id="post-list" class="flex-grow-1 px-xl-1">
  {% for post in posts %}
    <article class="card-wrapper card mb-4 post-item" data-category="{{ post.categories | join: ' ' }}">
      <a href="{{ post.url | relative_url }}" class="post-preview row g-0 flex-md-row-reverse">
        {% assign card_body_col = '12' %}

        {% if post.image %}
          {% assign src = post.image.path | default: post.image %}
          {% capture src %}{% include media-url.html src=src subpath=post.media_subpath %}{% endcapture %}
          {% assign alt = post.image.alt | xml_escape | default: 'Preview Image' %}
          {% assign lqip = null %}

          {% if post.image.lqip %}
            {% capture lqip_url %}{% include media-url.html src=post.image.lqip subpath=post.media_subpath %}{% endcapture %}
            {% assign lqip = 'lqip="' | append: lqip_url | append: '"' %}
          {% endif %}

          <div class="col-md-5">
            <img src="{{ src }}" alt="{{ alt }}" {{ lqip }}>
          </div>

          {% assign card_body_col = '7' %}
        {% endif %}

        <div class="col-md-{{ card_body_col }}">
          <div class="card-body d-flex flex-column">
            <h1 class="card-title my-2 mt-md-0">{{ post.title }}</h1>

            <div class="card-text content mt-0 mb-3">
              <p>{% include post-description.html %}</p>
            </div>

            <div class="post-meta flex-grow-1 d-flex align-items-end">
              <div class="me-auto">
                <i class="far fa-calendar fa-fw me-1"></i>
                {% include datetime.html date=post.date lang=lang %}

                {% if post.categories.size > 0 %}
                  <i class="far fa-folder-open fa-fw ms-3 me-1"></i>
                  <span class="categories">
                    {% for category in post.categories %}
                      {{ category }}{%- unless forloop.last -%},{%- endunless -%}
                    {% endfor %}
                  </span>
                {% endif %}
              </div>

              {% if post.pin %}
                <div class="pin ms-3">
                  <i class="fas fa-thumbtack fa-fw"></i>
                  <span>{{ site.data.locales[lang].post.pin_prompt }}</span>
                </div>
              {% endif %}
            </div>
          </div>
        </div>
      </a>
    </article>
  {% endfor %}
</div>

<!-- ⚙ JS for Filtering -->
<script>
  function filterByCategory(category) {
    const posts = document.querySelectorAll('.post-item');
    posts.forEach(post => {
      const categories = post.getAttribute('data-category').split(' ');
      if (category === 'all' || categories.includes(category)) {
        post.style.display = 'block';
      } else {
        post.style.display = 'none';
      }
    });
  }
</script>

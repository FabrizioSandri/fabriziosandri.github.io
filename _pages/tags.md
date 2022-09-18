---
layout: page
title: Tags
permalink: /tags/
nav: false
---

{% for tag in site.tags %}
  <div class="table-responsive">
    <table class="table table-sm table-borderless">
      {% assign t = tag | first %}
      {% assign posts = tag | last %}

      <h4><i class="fas fa-hashtag fa-sm"></i> {{ t | capitalize }}</h4>
      

      <article>
        {% for post in posts %}
        <tr>
          <th scope="row">{{ post.date | date: "%b %-d, %Y" }}</th>
          <td class="col-sm-9">
              <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
          </td>
        </tr>
        {% endfor %}
      </article>
    </table>
  </div>
{% endfor %}

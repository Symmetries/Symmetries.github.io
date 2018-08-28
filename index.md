---
layout: home
title: null
---
<h1> About </h1>
Diego Lopez is currently an undergraduate mathematics and computer science
student at McGill University. Their interests lie mostly at the intersection
of mathematics and computer science.

# Projects
{% for item in site.projects %}
  <h2><a href="{{ item.url }}">{{ item.title }}</a></h2>
  <p>
    {{ item.description }}. 

    {% if item.demo %}
      <a href="{{ item.demo }}">Demo</a>.
    {% endif %}

    {% if item.source %} 
      <a href="{{ item.source }}">Source Code </a>.
    {% endif %}
  </p>
{% endfor %}

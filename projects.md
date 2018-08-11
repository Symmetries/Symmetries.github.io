---
layout: page
title: Projects
permalink: projects
---
{% for item in site.projects %}
  <h1><a href="{{ item.url }}">{{ item.title }}</a></h1>
  <p> {{ item.description }}. <a href="{{ item.demo }}">Demo.</a> </p>
{% endfor %}

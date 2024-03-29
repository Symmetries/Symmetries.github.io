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
  <h2>{{ item.title }}</h2>
  <p>
    {{ item.description }}. 
    {% if item.url %}
      <a href="{{ item.url }}">More info</a>.
    {% endif %}
    {% if item.demo %}
      <a href="{{ item.demo }}">Demo</a>.
    {% endif %}

    {% if item.source %} 
      <a href="{{ item.source }}">Source Code</a>.
    {% endif %}
  </p>
{% endfor %}

# Friends

* [Anna Brandenberger](https://abrandenberger.github.io/)
* [Alex Iannantuono](https://alexander.iannantuono.org/)
* [Gabriela Moisescu-Pareja](https://gabrielamp.github.io/)
* [Maia Darmon](https://maiadd.github.io/)
* [Marcel Goh](https://marcelgoh.github.io/)
* [Mia Tran](https://mytran2111.github.io/)
* [Rosie Zhao](https://rosieyzh.github.io/website/)
* [Shereen Elaidi](https://shereenelaidi.github.io/)
* [Stephen Fay](https://dcxst.github.io/)

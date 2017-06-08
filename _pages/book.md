---
permalink: "/book/"
title: "Books by James Pogran"
layout: archive
author_profile: true
read_time: false
---

{% capture written_label %}'None'{% endcapture %}

<div class="grid__wrapper">
  {% for collection in site.collections %}
    {% for post in collection.docs %}
      {% unless collection.output == false or collection.label == "posts" %}
        {% include archive-single.html type="grid" %}
      {% endunless %}
    {% endfor %}
  {% endfor %}
</div>
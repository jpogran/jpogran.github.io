---
permalink: "/blog/"
layout: archive
author_profile: true
read_time: true
share: false
related: true
---

<h3 class="archive__subtitle">{{ site.data.ui-text[site.locale].recent_posts | default: "Recent Posts" }}</h3>

{% for post in site.categories.blog %}
  {% include archive-single.html %}
{% endfor %}

{% include paginator.html %}

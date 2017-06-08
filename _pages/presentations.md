---
permalink: "/presentations/"
layout: archive
author_profile: true
read_time: true
share: false
related: true
---

<div class="grid__wrapper">
{% for post in site.categories.presentation %}
    {% include archive-single.html type="grid" %}
{% endfor %}
</div>

{% capture written_label %}'None'{% endcapture %}
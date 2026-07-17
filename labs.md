---
layout: default
title: Labs
---

# Labs

{% for lab in site.labs %}
<div class="card mb-3">
  <div class="card-body">
    <h2 class="card-title fs-5">{{ lab.title }}</h2>
    <div class="d-flex flex-wrap gap-2">
      {% for download in lab.downloads %}
      <a class="btn btn-primary" href="{{ download.file | relative_url }}">{{ download.label }}</a>
      {% endfor %}
    </div>
  </div>
</div>
{% endfor %}

---
layout: default
title: Labs
permalink: /
---

{% for lab in site.labs %}
<div class="card mb-3">
  <div class="card-body">
    <h2 class="card-title fs-5">{{ lab.title }}</h2>
    {% if lab.session or lab.report_due %}
    <ul class="list-unstyled card-text text-body-secondary mb-3">
      {% if lab.session %}<li><strong>Session:</strong> {{ lab.session }}</li>{% endif %}
      {% if lab.report_due %}<li><strong>Report due:</strong> {{ lab.report_due }}</li>{% endif %}
    </ul>
    {% endif %}
    <div class="d-flex flex-wrap gap-2">
      <a class="btn btn-primary" href="{{ lab.url | relative_url }}">manual</a>
      {% if lab.manual_pdf %}
      <a class="btn btn-outline-primary" href="{{ lab.manual_pdf | relative_url }}">PDF</a>
      {% endif %}
      {% for download in lab.downloads %}
      <a class="btn btn-primary" href="{{ download.file | relative_url }}">{{ download.label }}</a>
      {% endfor %}
    </div>
  </div>
</div>
{% endfor %}

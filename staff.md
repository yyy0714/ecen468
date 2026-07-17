---
layout: default
title: Staff
---

# Staff

## Instructor

{% assign instructors = site.staff | where: 'role', 'Instructor' %}
{% for staffer in instructors %}
<div class="card mb-3">
  <div class="card-body">
    <h3 class="card-title fs-5">{{ staffer.name }}</h3>
    <ul class="list-unstyled card-text mb-0">
      <li><strong>E-mail:</strong> <a href="mailto:{{ staffer.email }}">{{ staffer.email }}</a></li>
      <li><strong>Office:</strong> {{ staffer.office }}</li>
      <li><strong>Office Hours:</strong> {{ staffer.office_hours }}</li>
    </ul>
  </div>
</div>
{% endfor %}

## Teaching Assistants

{% assign teaching_assistants = site.staff | where: 'role', 'Teaching Assistant' %}
{% for staffer in teaching_assistants %}
<div class="card mb-3">
  <div class="card-body">
    <h3 class="card-title fs-5">{{ staffer.name }}</h3>
    <ul class="list-unstyled card-text mb-0">
      <li><strong>E-mail:</strong> <a href="mailto:{{ staffer.email }}">{{ staffer.email }}</a></li>
      <li><strong>Office:</strong> {{ staffer.office }}</li>
      <li><strong>Office Hours:</strong> {{ staffer.office_hours }}</li>
    </ul>
  </div>
</div>
{% endfor %}

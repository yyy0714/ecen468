---
layout: default
title: Notes
---

# Notes

{% assign notes = site.notes | reverse %}
<div class="accordion" id="notesAccordion">
  {% for note in notes %}
  <div class="accordion-item">
    <h2 class="accordion-header">
      <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target="#note-{{ forloop.index }}" aria-expanded="false" aria-controls="note-{{ forloop.index }}">
        {{ note.title }}
      </button>
    </h2>
    <div id="note-{{ forloop.index }}" class="accordion-collapse collapse" data-bs-parent="#notesAccordion">
      <div class="accordion-body">
        {{ note.content | markdownify }}
      </div>
    </div>
  </div>
  {% endfor %}
</div>

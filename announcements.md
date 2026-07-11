---
layout: page
title: Announcements
nav_exclude: true
---

# Announcements

{% assign announcements = site.announcements | reverse %}
{% for announcement in announcements %}
{{ announcement }}
{% endfor %}

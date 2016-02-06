---
layout: default
title: Slides

---

{% for presentation in site.presentations %}
  {% include default_item_summary.html title=presentation.title summary=presentation.content metadata=presentation.date url=presentation.slides_url %}
  <hr />
{% endfor %}

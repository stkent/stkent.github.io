---
layout: default
title: Slides

---

{% for presentation in site.presentations %}
<summary>{% include default_item_summary.html title=presentation.title summary=presentation.content metadata=presentation.date url=presentation.slides_url %}</summary>
{% endfor %}

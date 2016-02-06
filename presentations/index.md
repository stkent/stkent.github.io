---
layout: default
title: Slides

---

{% for presentation in site.presentations %}
  {% capture formatted_presentation_date %}{{ presentation.date | date: "%B %Y" }}{% endcapture %}
  {% include default_item_summary.html title=presentation.title summary=presentation.content metadata=formatted_presentation_date url=presentation.slides_url %}
  <hr />
{% endfor %}

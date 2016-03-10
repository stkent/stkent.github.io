---
layout: default
title: Slides

---

{% for presentation in site.presentations %}
  {% capture formatted_presentation_date %}
    <i class="fa fa-calendar"></i> {{ presentation.date | date: "%B %Y" }}
  {% endcapture %}
  {% include talk_summary.html title=presentation.title summary=presentation.content metadata=formatted_presentation_date slides_url=presentation.slides_url %}
  <hr />
{% endfor %}

---
layout: default
title: Slides

---

{% for talk in site.talks %}
  {% capture formatted_talk_date %}
    <i class="fa fa-calendar"></i> {{ talk.date | date: "%B %Y" }}
  {% endcapture %}
  {% include talk_summary.html title=talk.title summary=talk.content metadata=formatted_talk_date slides_url=talk.slides_url %}
  <hr />
{% endfor %}

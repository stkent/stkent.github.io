---
layout: default
title: Talks

---

{% for talk in site.talks %}
  {% capture talk_display_date %}
    <i class="fa fa-calendar"></i> {{ talk.date | date: "%B %Y" }}
  {% endcapture %}
  {% include talk_summary.html talk=talk talk_display_date=talk_display_date %}
  <hr />
{% endfor %}

---
layout: default
title: Apps

---

{% for app in site.apps %}
{% capture full_app_title %}{{ app.title }} | {{ app.platform }}{% endcapture %}
<summary>{% include default_item_summary.html title=full_app_title summary=app.content metadata=app.date_range url=app.store_url %}</summary>
{% endfor %}

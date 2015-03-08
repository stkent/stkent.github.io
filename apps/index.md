---
layout: default
title: Apps

---

{% for app in site.apps %}
<summary>{% include default_item_summary.html title=app.title summary=app.content metadata=app.date_range url=app.store_url %}</summary>
{% endfor %}

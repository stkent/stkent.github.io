---
layout: default
title: Apps

---
{% for app in site.apps %}
  {% capture decorated_title %}{{ app.title }}{% if app.platform == "Android" %} <i class="fa fa-android"></i>{% endif %}{% endcapture %}
  {% capture decorated_date_range %}<i class="fa fa-calendar"></i> {{ app.date_range }}{% endcapture %}

  {% include default_item_summary.html
    relative_logo_url=app.relative_logo_url
    title=decorated_title
    metadata=decorated_date_range
    content=app.content
    action_url=app.store_url
    action_label="Store Page" %}
  <hr />
{% endfor %}

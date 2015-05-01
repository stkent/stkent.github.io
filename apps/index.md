---
layout: default
title: Apps

---

{% for app in site.apps %}
  <a name="{{ app.title | replace:' ','-' | downcase }}"></a>{% capture full_app_title %}{{ app.title }} | {{ app.platform }}{% endcapture %}
  <summary>
  {% include default_item_summary.html title=full_app_title summary=app.content metadata=app.date_range url=app.store_url %}
  </summary>
  <div class="image-container">
  {% for relative_img_url in app.relative_img_urls %}<img src="{{ site.baseurl }}{{ relative_img_url }}" width="40%" />{% endfor %}
  </div>
  <hr />
{% endfor %}

---
layout: default
title: Libraries

---

{% for library in site.libraries %}
  {% capture decorated_title %}{{ library.title }}{% if library.platform == "Android" %} <i class="fa fa-android"></i>{% endif %}{% endcapture %}

  {% include default_item_summary.html
    relative_logo_url=library.relative_logo_url
    title=decorated_title
    metadata=decorated_date_range
    content=library.content
    action_url=library.github_url
    action_label="GitHub Repo" %}
  <hr />
{% endfor %}

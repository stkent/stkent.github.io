---
layout: default
title: Apps

---
{% for app in site.apps %}
<!-- TODO: refactor this into a separate include -->
  <div style="display: flex; flex-wrap: nowrap; align-items: center">
    <img src="{{ app.relative_logo_url }}" width="25%" style="max-width: 125px" />
    <div style="flex-grow: 1; margin-left: 16px">
      <h1>{{ app.title }} <i class="fa fa-android"></i></h1> <!-- TODO: icon should be conditional -->
      <div class='metadata'>
        <i class="fa fa-calendar"></i> {{ app.date_range }}<br />
      </div>
    </div>
  </div>
  {{ app.content }}
  <a href="{{ app.store_url }}" class="btn">Store Page</a>
  <hr />
{% endfor %}

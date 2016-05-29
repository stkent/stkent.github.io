---
layout: default
title: Apps

---

{% for app in site.apps %}
  <a name="{{ app.title | replace:' ','-' | downcase }}"></a>
  {% capture full_app_title %}
    {{ app.title }} | {{ app.platform }}
  {% endcapture %}
  {% capture formatted_date_range %}
    <i class="fa fa-calendar"></i> {{ app.date_range }}
  {% endcapture %}
  {% include default_item_summary.html title=full_app_title summary=app.content metadata=formatted_date_range url=app.store_url %}
  <div class="image-container">
  	{% for relative_img_url in app.relative_img_urls %}<img src="{{ site.baseurl }}{{ relative_img_url }}" width="40%" />{% endfor %}
  </div>
  <hr />
{% endfor %}

{% include jquery.html %}

<script type = "text/javascript">
   $(function(){
      $("p").hide()
   });
</script>

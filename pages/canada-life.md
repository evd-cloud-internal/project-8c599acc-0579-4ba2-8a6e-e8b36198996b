---
name: Canada Life
assetId: 8ff8f5e2-a3f6-4691-b0c9-d93c66e69124
type: page
---

---
reporting_block_id: canadalife
---
# {{$translations.client_report_title.canadalife}} {% .text-center  %}

<!-- # Client Report - Canada Life  {% .text-center  %} -->

---

{% row align="center"%}
    {% partial file="filters/partnerspecificfilters/clfilterset" /%}
    {% partial file="filters/datefilters" /%}
{% /row %}
---

{% partial file="dashboardlayout"
    variables={
        reporting_block_id = $reporting_block_id
    } 
/%}

<!-- Do not make any layout changes to this page. Do everything in this partial ^^ -->

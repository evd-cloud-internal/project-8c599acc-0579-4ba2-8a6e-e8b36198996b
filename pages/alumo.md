---
name: Alumo
assetId: 72207fee-5cd0-4d73-a6ed-84514b96ec6c
type: page
---

---
reporting_block_id: student-solutions
---

<!-- # Client Report - Alumo  {% .text-center  %} -->
# {{$translations.client_report_title.alumo}} {% .text-center  %}
---

{% row align="center"%}
    {% partial file="filters/partnerspecificfilters/ssfilterset" /%}
    {% partial file="filters/datefilters" /%}
{% /row %}
---

{% partial file="dashboardlayout"
    variables={
        reporting_block_id = $reporting_block_id
    } 
/%}

<!-- Do not make any layout changes to this page. Do everything in this partial ^^ -->
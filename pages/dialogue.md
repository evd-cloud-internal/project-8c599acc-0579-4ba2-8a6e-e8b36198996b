---
name: Dialogue
assetId: 4cbf7145-f9f6-4b48-a4b3-0d81a40a89c7
type: page
---

---
reporting_block_id: dia
---

<!-- # Client Report - Dia  {% .text-center  %} -->
# {{$translations.client_report_title.dia}} {% .text-center  %}

{% row %}

    {% partial file="filters/partnerspecificfilters/diafilterset"/%}
    {% partial file="filters/datefilters" /%}
{% /row %}


{% partial file="dashboardlayout"
    variables={
        reporting_block_id = $reporting_block_id
    } 
/%}

<!-- Do not make any layout changes to this dashout. Do everything in this partial ^^ -->
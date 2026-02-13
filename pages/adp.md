---
name: ADP
assetId: b264780d-c86e-4942-a80b-b595abd2ec16
type: page
---

---
reporting_block_id: ADP
---

<!-- # Client Report - Dia  {% .text-center  %} -->
# {{$translations.client_report_title.adp}} {% .text-center  %}

{% row %}

    {% partial file="filters/partnerspecificfilters/adpfilterset"
        variables={
            reporting_block_id = $reporting_block_id
        } 
    /%}
    {% partial file="filters/datefilters" /%}
{% /row %}


{% partial file="dashboardlayout"
    variables={
        reporting_block_id = $reporting_block_id
    } 
/%}

<!-- Do not make any layout changes to this dashout. Do everything in this partial ^^ -->

---
name: Equitable Life
assetId: 7f78ab16-bf03-4f69-91f2-0cdc0f2385ce
type: page
---

---
reporting_block_id: Equitable Life
---

<!-- # Client Report - Dia  {% .text-center  %} -->
# {{$translations.client_report_title.eq_life}} {% .text-center  %}

{% row %}

    {% partial file="filters/partnerspecificfilters/eqfilterset"
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


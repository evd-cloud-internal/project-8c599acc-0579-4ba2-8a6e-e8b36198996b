---
name: Sun Life (BDR)
assetId: 4bf3d239-fc5f-4fc5-bb4e-0b31db467cd6
type: page
---

---
reporting_block_id: sunlife
include_block_benchmark: false
include_industry_benchmark: false
palette: ["blue","red", "black", "purple", "green"]
---

<!-- # Client Report - Sun Life  {% .text-center  %} -->
# {{$translations.client_report_title.sunlife}} {% .text-center  %}
---


{% row align="center"%}
    {% partial file="filters/partnerspecificfilters/slbdrfilterset" /%}
    {% partial file="filters/datefilters" /%}
{% /row %}
---

{% partial file="dashboardlayout"
    variables={
        reporting_block_id = $reporting_block_id
        include_block_benchmark = $include_block_benchmark
        include_industry_benchmark = $include_industry_benchmark
    } 
/%}

<!-- Do not make any layout changes to this page. Do everything in this partial ^^ -->

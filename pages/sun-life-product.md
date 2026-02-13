---
name: Sun Life (Product)
assetId: f6815f2c-c4a8-461c-aeb5-12e54400c9f8
type: page
---

---
reporting_block_id: sunlife
include_industry_benchmark: false
palette: ["blue","red", "black", "purple", "green"]
---

<!-- # Client Report - Sun Life  {% .text-center  %} -->
# {{$translations.client_report_title.sunlife}} {% .text-center  %}
---


{% row align="center"%}
    {% partial file="filters/partnerspecificfilters/slproductfilterset" /%}
    {% partial file="filters/datefilters" /%}
{% /row %}
---

{% partial file="dashboardlayout"
    variables={
        reporting_block_id = $reporting_block_id
        include_industry_benchmark = $include_industry_benchmark
    } 
/%}

<!-- Do not make any layout changes to this page. Do everything in this partial ^^ -->
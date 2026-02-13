---
name: Embedded_Presto
assetId: 246e0954-113e-4286-a4fd-3f4209f810e3
type: page
---

---
reporting_block_id: dia
---

{% row %}
    {% partial file="filters/partnerspecificfilters/embeddedfilterset" /%}
    {% partial file="filters/datefilters" /%}
{% /row %}
---

{% partial file="dashboardlayout"
    variables={
        reporting_block_id = $reporting_block_id
    } 
/%}

<!-- Do not make any layout changes to this dashboard. Do everything in this partial ^^ -->

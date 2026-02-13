---
name: AI_metric_buckets_positive
assetId: 0d7df2a0-9dfc-4c85-801b-945331ca6e22
type: partial
---

```sql table_inner_join
select
organization_id
, organization_name 
from {{filtered_table}}
```


```sql metric_calc_ai_feature_positive
with cte as (
select
  hackathon_buckets.*
  , filt_joins.organization_name
from hackathon_buckets
inner join (select distinct organization_id, organization_name from {{table_inner_join}}) as filt_joins
  on toString(filt_joins.organization_id) = hackathon_buckets.org_id
where sentiment = 'positive'
)

select
bucket
, org_id
, response_count
, toFloat64(response_count) / sum(response_count) over () as response_pct
from cte
order by response_count desc
limit 3
```


{% table
  data="metric_calc_ai_feature_positive"
  title="positive feedback"
%}
    {% dimension value="bucket" title="Bucket" /%}
    {% measure value="response_count as Responses" title="Response Count" fmt="#,##0" /%}
    {% measure value="response_pct" title="Response %" fmt="pct1" /%}
{% /table %}

{% horizontal_bar_chart
  data="metric_calc_ai_feature_positive"
  x="response_pct"
  y="bucket"
  data_labels={
    position="right"
    fmt="pct1"
  }
/%}
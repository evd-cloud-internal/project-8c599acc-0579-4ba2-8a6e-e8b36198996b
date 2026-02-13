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
with filt_respsonses as (
select
*
from hackathon_buckets
where created_at >= '2026-02-13 15:50:38.345'
)

,cte as (
select
  filt_respsonses.*
  , filt_joins.organization_name
from filt_respsonses
inner join (select distinct organization_id, organization_name from {{table_inner_join}}) as filt_joins
  on toString(filt_joins.organization_id) = filt_respsonses.org_id
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
  info="AI bucketed summary of positive feedback from PJS surveys when asked If you had a magic wand, what is one thing would you wish we could help you with and how?"
%}
    {% dimension value="bucket" title="Bucket" /%}
    {% measure value="response_pct as percentage_of_respsonses" fmt="pct1" /%}
{% /table %}


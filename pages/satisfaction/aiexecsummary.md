---
name: AI_exec_summary
assetId: 63dd43e6-b7d3-44e2-babe-913470f20085
type: partial
---

```sql table_inner_join
select
organization_id
, organization_name 
from {{filtered_table}}
```


```sql metric_calc_ai_exec_summary
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
where sentiment = 'neutral'
)

select
bucket
from cte
limit 1

```



{% table
  data="metric_calc_ai_exec_summary"
  title="executive summary"
  info="AI exec summary"
%}
    {% dimension value="bucket" title="Bucket" wrap=true /%}
{% /table %}


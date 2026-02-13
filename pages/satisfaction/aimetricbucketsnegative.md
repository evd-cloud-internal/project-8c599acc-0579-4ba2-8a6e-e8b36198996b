---
name: AI_metric_buckets_negative
assetId: 92537556-58fd-40e5-a903-1a3c855ae969
type: partial
---

```sql table_inner_join_neg
select
organization_id
, organization_name 
from {{filtered_table}}
```


```sql metric_calc_ai_feature_negative
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
where sentiment = 'negative'
)

select
bucket
, org_id
, response_count
, toFloat64(response_count) / sum(response_count) over () as response_count_pct
from cte
order by response_count desc
limit 2
```

```sql multi_org_select_ai
select
count(distinct organization_id) as count_orgs
from {{table_inner_join_neg}}
```


{% if
  data="multi_org_select_ai"
  condition="has_rows"
  where="count_orgs > 1"
%}
  
  {% callout type="info" %}
  On a **block** level, this is what your members would like to see:
  {% /callout %}
  {% line_break lines=1 /%}
  {% table
    data="metric_calc_ai_feature_negative"
    title="negative feedback"
    show_total_row=false
    info="AI bucketed summary of negative feedback from PJS surveys when asked If you had a magic wand, what is one thing would you wish we could help you with and how?"
  %}
    {% dimension value="bucket" /%}
    {% measure value="sum(response_count_pct) as Percentage_of_responses" fmt="pct1" /%}
  {% /table %}


{% /if %}
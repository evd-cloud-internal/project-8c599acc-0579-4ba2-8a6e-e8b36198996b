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
with cte as (
select
  hackathon_buckets.*
  , filt_joins.organization_name
from hackathon_buckets
inner join (select distinct organization_id, organization_name from {{table_inner_join}}) as filt_joins
  on toString(filt_joins.organization_id) = hackathon_buckets.org_id
where sentiment = 'negative'
)

select
bucket
, org_id
, response_count
, toFloat64(response_count) / sum(response_count) over () as response_count_pct
from cte
order by response_count desc
limit 3
```


{% table
  data="metric_calc_ai_feature_negative"
  title="negative feedback"
  show_total_row=false
  info="AI bucketed summary of negative feedback from PJS surveys when asked If you had a magic wand, what is one thing would you wish we could help you with and how?"
%}
  {% dimension value="bucket" /%}
  {% measure value="sum(response_count) as Responses" fmt="#,##0" /%}
  {% measure value="sum(response_count_pct) as Percentage" fmt="pct1" /%}
{% /table %}

---
name: WIP_challenges_top_table
assetId: 81e90b12-431c-46de-8a06-ffa53d0a97b0
type: partial
---

---
privacy_restriction_value: 5
reporting_block_id: dia
---
{% partial
    file="combinefilters"
/%}

```sql metric_desc_challenges_reach
select
    value as description
from base_client_reporting_metric_descriptions
where metric_name = 'Challenges Reach'
    and language = 'EN'
```

```sql metric_calculation_challenges_top_table
select 
challenge_name,
count(distinct user_id) as users_joined

from wellness_activities_with_contracts
where date_month_est >= {{date_start.selected}}
and date_month_est <= {{date_end.selected}}
and {{filter_select.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
and date_month_est < date_trunc('month',today())
and activity = 'challenge_joined'
group by 1
order by 2 desc
```
{% table
    data="metric_calculation_challenges_top_table"
%}
{% /table %}

```sql restriction_challenges_top_table
select
users_joined
, challenge_reach
from {{metric_calculation_challenges_reach}} as metric_calculation
where users_joined >= {{$privacy_restriction_value}}
```

{% if
    data="restriction_challenges_reach"
    condition="has_rows"
%}

<!-- {% row card=true %} -->
{% big_value
    data="restriction_challenges_reach"
    value="challenge_reach"
    title="Challenges Reach"
    fmt="pct"
    info="The percentage of active users that joined at least one challenge."
    /%}

{% /if %}

{% else %}

{% callout
type="warning"
%}
Metric value(s) cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else %}
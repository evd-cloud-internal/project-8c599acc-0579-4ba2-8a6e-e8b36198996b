---
name: challenges reach
assetId: f9a3d5e6-3ea9-4ae9-bdcd-507a51f34637
type: partial
---

```sql metric_calculation_challenges_reach
select count(distinct
    case when activity = 'challenge_joined' then user_id end) as users_joined
    , count(distinct
    case when activity = 'challenge_joined' then user_id end) /
    nullIf(count(distinct user_id), 0) as challenge_reach
from wellness_activities_with_contracts_truncated
where date_month_est >= {{date_start.selected}}
and date_month_est <= {{date_end.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
and date_month_est < date_trunc('month',today())
```


```sql restriction_challenges_reach
select
users_joined
, challenge_reach
from {{metric_calculation_challenges_reach}} as metric_calculation
where users_joined >= {{$privacy_restriction_low}}
```

{% if
    data="restriction_challenges_reach"
    condition="has_rows"
%}

<!-- {% row card=true %} -->
{% big_value
    data="restriction_challenges_reach"
    value="challenge_reach"
    title="{{$translations.metric_definitions.utilization.challenges_reach.title}}"
    fmt="pct"
    info="{{$translations.metric_definitions.utilization.challenges_reach.description}}"
    info_link="#{{$translations.metric_definitions.utilization.challenges_reach.info_link}}"
    info_link_title="{{ $translations.read_more}}"
    /%}

{% /if %}

{% else %}

{% callout
    type="warning"
    %}
    
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.challenges_reach.title}}{{$translations.privacy_disclaimer_end}}

{% /callout %}

{% /else %}


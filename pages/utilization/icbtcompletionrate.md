---
name: icbt_completion_rate
assetId: 0a5dd99f-d94d-4ab7-80c4-8cb4de6c4a70
type: partial
---

```sql metric_desc_icbt_completion_rate
select
    value as description
from base_client_reporting_metric_descriptions
where metric_name = 'iCBT Completion Rate'
    and language = 'EN'
```

```sql metric_calculation_icbt_completion_rate
select avg(completion_rate) as completion_rate
    , count(distinct user_interval_id) as number_users
from icbt_independent_activity_with_contracts_truncated
where {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and last_activity_at_month >= {{date_start.selected}}
    and last_activity_at_month <= {{date_end.selected}}
```


```sql restriction_icbt_completion_rate
select *
from {{metric_calculation_icbt_completion_rate}} as metric_calculation
where number_users >= {{$privacy_restriction_low}}
```

{% if
    data="restriction_icbt_completion_rate"
    condition="has_rows"
%}

{% big_value
    data="restriction_icbt_completion_rate"
    value="completion_rate as _"
    title="{{$translations.metric_definitions.utilization.icbt_completion_rate.title}}"
    fmt="pct"
    info="{{$translations.metric_definitions.utilization.icbt_completion_rate.description}}"
    info_link="#{{$translations.metric_definitions.utilization.icbt_completion_rate.info_link}}"
    info_link_title="{{ $translations.read_more}}"
    /%}

{% /if %}

{% else %}

{% callout
    type="warning"
    %}

    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.icbt_completion_rate.title}}{{$translations.privacy_disclaimer_end}}

{% /callout %}

{% /else %}


---
name: OLD_monthly_care_sessions
assetId: 3921f48f-c13d-402d-b8b5-4e2098a08049
type: partial
---

```sql desc_monthly_care_sessions
select
    value as description
from base_client_reporting_metric_descriptions
where language = 'EN'
    and metric_name = 'Monthly Primary Care Sessions'
```

```sql metric_calculation_monthly_care_sessions
with care_sessions as (
select date_month
    , sum(sessions_primary_care_utilization_rate) as care_sessions
from client_reporting_monthly
where date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and date_month < date_trunc('month',today())
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
)

, ranked as (
select date_month
    , care_sessions
    , sum(care_sessions) over (
        partition by year(date_month)
        order by date_month
        rows unbounded preceding
    ) as care_sessions_ytd
    , lag(care_sessions) over (
        partition by month(date_month)
        order by date_month
    ) as care_sessions_previous_year
    , sum(care_sessions) over () as cumulative_sessions
    , rank() over (order by date_month desc) as rank
from care_sessions
)
select
*
from ranked
where rank <=12

```

```sql restrictions_monthly_care_sessions
with privacy_restriction_value as (
select
privacy_restriction_metric_value as value
from base_client_reporting_metric_privacy_restrictions
where privacy_restriction_metric_name = 'cases'
)

select
date_month
, case
    when care_sessions >= privacy_restriction_value.value then care_sessions
    else null
end as care_sessions
, case
    when care_sessions_previous_year >= privacy_restriction_value.value then care_sessions_previous_year
    else null
end as care_sessions_previous_year
, case
    when care_sessions_ytd >= privacy_restriction_value.value then care_sessions_ytd
    else null
end as care_sessions_ytd
from {{metric_calculation_monthly_care_sessions}} as metric_calculation
cross join privacy_restriction_value
```

```sql full_restrictions_check_monthly_care_sessions
select
care_sessions
, care_sessions_previous_year
, care_sessions_ytd
from {{restrictions_monthly_care_sessions}}
where (care_sessions is not null) or (care_sessions_previous_year is not null) or (care_sessions_ytd is not null)
```


{% if
    data="full_restrictions_check_monthly_care_sessions"
    condition="has_rows"
%}

{% row card=true %}

{% combo_chart
    data="restrictions_monthly_care_sessions"
    x="date_month"
    date_grain="month"
    title="Monthly Care Sessions"
    x_axis_options={
        title=""
    }
    info=""
%}
    {% bar
        y="care_sessions"
        data_labels={
            position="above"
        }
    /%}
    {% bar
        y="care_sessions_previous_year"
        axis="y2"
        data_labels={
            position="above"
        }
    /%}
    {% line
        y="care_sessions_ytd as Session_cumulative"
        data_labels={
            position="above"
        }
        axis="y2"
    /%}    
{% /combo_chart %}

{% /row %}

{% /if %}

{% else_if
    data="full_restrictions_check_monthly_care_sessions"
    condition="no_rows"
%}

{% callout
type="warning"
%}
Monthly Care Sessions cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else_if %}


<!-- problem is the metric each require their own individual restrictions. 
need case when logic per month. -->




---
name: OLD_monthly_utilization_rate
assetId: 3f9a8afe-adf7-4ac7-a62c-874245cd24bd
type: partial
---

---
privacy_restriction_value: 5
---

```sql desc_monthly_utilization_rate
select
    value as description
from base_client_reporting_metric_descriptions
where language = 'EN'
    and metric_name = 'Monthly Utilization Rate'
```

```sql metric_calculation_monthly_utilization_rate
with aliases as (
select
    date_month
    , reporting_group_id
    , organization_id
    , sessions_primary_care as cases_primary_care
    , cases_mental_health
    , cases_eap
    , avg_eligible_members_primary_care
    , avg_eligible_members_mental_health
    , avg_eligible_members_eap
from client_reporting_monthly
where date_month < date_trunc('month',today())
and date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)

, monthly as (
select
    date_month
    , sum(coalesce(aliases.cases_{{program_filter.literal}}, 0)) as cases
    , sum(coalesce(aliases.avg_eligible_members_{{program_filter.literal}}, 0)) as avg_eligible_members
from aliases
group by 1
)

, ranked as (
select date_month
    , divideDecimal(cases, avg_eligible_members, 3) as utilization_rate
    , cases
    , avg_eligible_members
    , rank() over (order by date_month desc) as rank
from monthly
)

select
*
from ranked
where rank <= 12
order by rank desc
```

```sql restrictions_monthly_utilization_rate
select
date_month
, cases
, utilization_rate
from {{metric_calculation_monthly_utilization_rate}} as metric_calculation
where cases >= {{$privacy_restriction_value}}
```


{% if
    data="restrictions_monthly_utilization_rate"
    condition="has_rows"
%}

    {% bar_chart
        data="restrictions_monthly_utilization_rate"
        x="date_month"
        date_grain="month"
        y="utilization_rate as _"
        info="Primary Care: The number of sessions divided by the average number of eligible members under the Primary Care program, within a given month. Mental Health+ or EAP: The number of cases divided by the average number of eligible members under the Mental Health+ or EAP programs, within a given month."
        y_fmt="pct1"
        title="Monthly Utilization Rate"
        data_labels={position="middle"}
        x_axis_options={
            min_interval="month"
            gridlines=false
            title=""
        }
        x_fmt="mmmm"
        y_axis_options={
            gridlines=false
        }
        info_link="#monthly-utilization-rate"
        info_link_title="Read more"
    %}
    {% /bar_chart %}

{% /if %}

{% else %}

{% callout
type="warning"
%}
Monthly Utilization Rate cannot be displayed because it is below privacy threshold
{% /callout %}
{% /else %}

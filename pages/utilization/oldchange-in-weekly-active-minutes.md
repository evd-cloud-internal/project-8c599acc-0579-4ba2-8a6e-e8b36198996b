---
name: old_change-in-weekly-active-minutes
assetId: d3450a5c-69ab-4b6e-9fc9-0137b6fdf826
type: partial
---

---
privacy_restriction_value: 5
---

```sql metric_calculation_change_in_minutes
with base_model as (
SELECT 
'1. Before Joining' as period 
, cohorts_before_joined as cohorts
, user_id
FROM wellness_active_minutes_steps_reporting_monthly_truncated
WHERE cohorts_before_joined is not null
and activity_month >= {{date_start.selected}}
and activity_month <= {{date_end.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
and activity_month < date_trunc('month',today())

union all

SELECT 
'2. After Joining' as period 
,cohorts_after_joined as cohorts
, user_id
FROM wellness_active_minutes_steps_reporting_monthly_truncated
where activity_month >= {{date_start.selected}}
and activity_month <= {{date_end.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
and activity_month < date_trunc('month',today())
)

, agg as (
select
period
, cohorts
, count(distinct user_id) as users
from base_model
group by 1,2
)

select
period
, cohorts
, users
, users / sum(users) over (partition by cohorts) as metric_value
from agg

```

{% if
    data="metric_calculation_change_in_minutes"
    condition="has_rows"
    where="users >= {{$privacy_restriction_value_low}}"
    %}

    {% combo_chart
        data="metric_calculation_change_in_minutes"
        x="cohorts"
        series="period"
        y_fmt="pct"
        x_axis_options={
            gridlines=false
            label_rotate=45
            title=""
        }
        y_axis_options={
            gridlines=false
        }
        title="Change in Weekly Active Minutes"
        info="The distribution of users based on their physical activity data for 30 days prior to joining and for the most recent 30 days in the selected time period after joining."
        chart_options={
            top_padding=5
        }
    %}
    {% bar
        y="metric_value as _"
        data_labels={
            position="above"
            fmt="pct1"
        }
    /%}
    {% /combo_chart %}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    Change in Weekly Active Minutes cannot be displayed because it is below privacy threshold
    {% /callout %}

{% /else %}


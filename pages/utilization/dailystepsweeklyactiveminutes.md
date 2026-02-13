---
name: daily_steps_weekly_active_minutes
assetId: bbdb378e-1151-4f68-81c6-2dfb9faaaf7c
type: partial
---

```sql metric_calculation_daily_steps_weekly_minutes
select activity_month as month
    , sum(steps)/max(nb_day_month)/count(distinct user_id) as average_daily_steps
    , 7*sum(active_minutes)/max(nb_day_month)/count(distinct user_id) as average_weekly_active_minutes
    , count(distinct user_id) as unique_users
from wellness_active_minutes_steps_reporting_monthly_truncated
where activity_month between {{date_start.selected}} and {{date_end.selected}}
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and activity_month < date_trunc('month',today())
    and period = '2. after joining' 
group by 1
order by 1 desc
```

```sql restrictions_daily_steps_weekly_minutes
select month
    , average_daily_steps
    , average_weekly_active_minutes
    , unique_users
from {{metric_calculation_daily_steps_weekly_minutes}}
where unique_users >= {{$privacy_restriction_low}}
```

{% stack card=true %}

    ### {{$translations.section_descriptions.utilization.physical_activity.title}}
    <!-- Physical Activity -->
    <!-- Tracks the average daily steps and weekly active minutes of the members that synced their tracker with the Dialogue app. -->
    {{$translations.section_descriptions.utilization.physical_activity.overview}}

{% if
    data="restrictions_daily_steps_weekly_minutes"
    condition="has_rows"
    %}

    {% combo_chart
        data="restrictions_daily_steps_weekly_minutes"
        x="month"
        x_axis_options={
            gridlines=false
            label_rotate=45
        }
        y_axis_options={
            gridlines=false
        }
        title="{{$translations.metric_definitions.utilization.daily_steps_weekly_minutes.title}}"
        info="{{$translations.metric_definitions.utilization.daily_steps_weekly_minutes.description}}"
        info_link="#{{$translations.metric_definitions.utilization.daily_steps_weekly_minutes.info_link}}"
        info_link_title="{{ $translations.read_more}}"
        chart_options={
            top_padding=6
        }
    %}
    {% bar
        y="average_daily_steps as `{{$translations.metric_definitions.utilization.daily_steps_weekly_minutes.average_daily_steps}}`"
        data_labels={
            position="above"
            fmt="num0"
        }
    /%}
    {% bar
        y="average_weekly_active_minutes as `{{$translations.metric_definitions.utilization.daily_steps_weekly_minutes.average_weekly_active_minutes}}`"
        axis="y2"
        data_labels={
            position="above"
            fmt="num0"
        }
    /%}
    {% line
        y="unique_users as `{{$translations.metric_definitions.utilization.daily_steps_weekly_minutes.unique_users}}`"
        data_labels={
            position="above"
            fmt="num0"
            border_color="#00000000"
        }
    /%}
    {% /combo_chart %}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.daily_steps_weekly_minutes.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}

{% /stack %}
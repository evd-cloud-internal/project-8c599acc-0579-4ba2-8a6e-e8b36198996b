---
name: monthly_utilization_rate_yoy
assetId: 53845fb7-c32f-4be8-a9a7-43acdb6c0b82
type: partial
---

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
where {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
and (
    date_month between {{date_start.selected}} and {{date_end.selected}} 
    or 
    date_month between addMonths({{date_start}}, -12) and addMonths({{date_end}}, -12)
    )
),
agg as (select
    date_month
    , max(toYear(date_month)) as year
    , sum(coalesce(aliases.avg_eligible_members_{{program_filter.literal}}, 0)) as avg_eligible_members
    , sum(coalesce(aliases.cases_{{program_filter.literal}}, 0)) as cases
    --, divideDecimal(cases, avg_eligible_members, 3) as utilization_rate
    , divideOrNull(toFloat32(cases), toFloat32(avg_eligible_members)) as utilization_rate
from aliases
group by 1
)

select
date_month
, year
, cases
, case when cases >= {{$privacy_restriction_low}}
then utilization_rate
else null
end as utilization_rate
from agg
where cases >= {{ $privacy_restriction_low }}
```

```sql monthly_util_rate_all_programs_selected
select
{{program_filter.selected}} as program_selection
```


{% if
    data="monthly_util_rate_all_programs_selected"
    condition="has_rows"
    where="program_selection = 'all'"
    %}

    {% callout
        type="info"
        %}
        <!-- Monthly Utilization Rate is only available for **individual** programs -->
        {{$translations.privacy_disclaimer_start}} 
{{$translations.metric_definitions.utilization.monthly_utilization_rate.title}}{{$translations.all_program_restrictions}}
    {% /callout %}

{% /if %}

{% else_if
    data="metric_calculation_monthly_utilization_rate"
    condition="has_rows"
    where="date_month between {{date_start.selected}} and {{date_end.selected}}"
    %}

    {% bar_chart
        data="metric_calculation_monthly_utilization_rate"
        x="date_month"
        y="utilization_rate as _"
        info="{{$translations.metric_definitions.utilization.monthly_utilization_rate.description}}"
        y_fmt="pct1"
        title="{{$translations.metric_definitions.utilization.monthly_utilization_rate.title}}"
        data_labels={position="middle" border_color="#00000000"}
        x_axis_options={
            min_interval="month"
            gridlines=false
            title=""
        }
        x_fmt="mmm"
        y_axis_options={
            gridlines=false
        }
        info_link="#{{$translations.metric_definitions.utilization.monthly_utilization_rate.info_link}}"
        info_link_title="{{ $translations.read_more}}"
        series="year"
        stacked=false
        date_grain="month of year"
        order="year asc"
    %}
    {% /bar_chart %}

{% /else_if %}

{% else %}

    {% callout
        type="warning"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.monthly_utilization_rate.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}
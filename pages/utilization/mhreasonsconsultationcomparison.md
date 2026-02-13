---
name: mh_reasons_consultation_comparison
assetId: abe40832-5a84-46ad-9631-2a3a428ce08a
type: partial
---

```sql metric_calculation_mh_reason_for_consult
with episodes_with_contracts_program_mapping as (
select
    case
        when {{program_filter.selected}} = 'all' then 'all'
        else program
    end as program
    , *
from episodes_with_contracts
where program is not null
and program not in ('wellness', 'mso')   
),
reasons_for_consult as (
select
  mh_reason_for_consult_{{ $translations.inline_query}} as mh_reason_for_consult
  , program

    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then episode_id end) as cases_current

    , case when 
    count(distinct case when date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then episode_id end) >= {{$privacy_restriction_medium}}
    then count(distinct case when date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then episode_id end) 
    else null
    end as cases_previous

    , sum(cases_current) over () as total_cases_current

    , sum(cases_previous) over () as total_cases_previous

from episodes_with_contracts_program_mapping
where (date_month_est between {{date_start.selected}}
    and {{date_end.selected}}
    or date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12))
    and program = {{program_filter.selected}}
    and is_suitable_episode
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and mh_reason_for_consult_{{ $translations.inline_query}} is not null
group by mh_reason_for_consult_{{ $translations.inline_query}}, program
)
select
    mh_reason_for_consult
    , program
    , cases_current / total_cases_current as proportion_current
    , cases_previous / total_cases_previous as proportion_previous
    , cases_current
    , cases_previous
    , total_cases_current
    , total_cases_previous
from reasons_for_consult
```


```sql restrictions_mh_reason_for_consult
select
    mh_reason_for_consult
    , program
    , proportion_current
    , total_cases_current
    , cases_current
    , proportion_previous
    , total_cases_previous
    , cases_previous
from {{metric_calculation_mh_reason_for_consult}} as metric_calculation
where cases_current >= {{$privacy_restriction_medium}}
```

{% if
    data="restrictions_mh_reason_for_consult"
    condition="has_rows"
    %}

    {% table
        data="restrictions_mh_reason_for_consult"
        order="cases_current desc"
        title="{{$translations.metric_definitions.utilization.mh_reason_for_consult.title}}"
        info="{{$translations.metric_definitions.utilization.mh_reason_for_consult.description}}"
        info_link="#{{$translations.metric_definitions.utilization.mh_reason_for_consult.info_link}}"
        info_link_title="{{ $translations.read_more}}"
    %}
    {% dimension
        value="mh_reason_for_consult"
        title="{{$translations.metric_definitions.utilization.mh_reason_for_consult.mental_health_reason}}"
    /%}

    {% dimension
        value="cases_current"
        title="{{$translations.metric_definitions.utilization.mh_reason_for_consult.cases}}"
        fmt="num"
    /%}
    {% measure
        value="cases_current"
        comparison={
            compare_vs="target"
            target="cases_previous"
            pct_fmt="pct1"
        }
        title=""
        viz="delta"
    /%}
    {% dimension
        value="proportion_current"
        fmt="pct1"
        title="{{$translations.metric_definitions.utilization.mh_reason_for_consult.percent_cases}}"
        info="{{$translations.metric_definitions.utilization.mh_reason_for_consult.percent_cases_description}}"
    /%}
    {% measure
    value="proportion_current"
    fmt="pct1"
    comparison={
        compare_vs="target"
        target="proportion_previous"
        pct_fmt="pct1"
    }
    title=""
    viz="delta"
    /%}
    {% /table %}

    <!-- {% bar_chart
        data="restrictions_mh_reason_for_consult"
        x="proportion_current"
        y="mh_reason_for_consult"
        x_fmt="pct"
        title="Reasons for Mental Health Consultations"
        order="proportion_current asc"
        info="The breakdown of reasons for mental health consultations, as indicated by the mental health specialist at initial assessment. Please note that due to the nature of sleep issues, the percentages reported may be lower for clients enrolled in our Primary Care program."
        y_axis_options={
            gridlines=false
            title=""
        }
        x_axis_options={
            label_wrap=true
            gridlines=false
            title=""
            }
    /%} -->

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.mh_reason_for_consult.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}
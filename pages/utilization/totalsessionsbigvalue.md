---
name: total_sessions_big_value
assetId: 477959a9-6c98-414b-9635-51b66986f20c
type: partial
---

```sql metric_calculation_total_sessions
with client_report_time as (
select 
    sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then sessions end) as sessions_all_current_year
    , sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then sessions end) as sessions_all_last_year

    , sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then sessions_eap end) as sessions_eap_current_year
    , sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then sessions_eap end) as sessions_eap_last_year

    , sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then sessions_mental_health end) as sessions_mental_health_current_year
    , sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then sessions_mental_health end) as sessions_mental_health_last_year

    , sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then sessions_primary_care_utilization_rate end) as sessions_primary_care_current_year
    , sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then sessions_primary_care_utilization_rate end) as sessions_primary_care_last_year

from client_reporting_monthly
where {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and date_month <= {{date_end.selected}}
    and date_month >= date_trunc('month',addMonths({{date_start.selected}}, -12))
)
, program_specific_values as (
select
sessions_{{program_filter.literal}}_current_year
, case
    when sessions_{{program_filter.literal}}_last_year < {{$privacy_restriction_low}} then null
    else sessions_{{program_filter.literal}}_last_year
end as sessions_{{program_filter.literal}}_last_year
from client_report_time
)

select
sessions_{{program_filter.literal}}_current_year as sessions
, sessions_{{program_filter.literal}}_last_year as sessions_last_year
from program_specific_values
```

{% if
    data="metric_calculation_total_sessions"
    where="sessions >= {{$privacy_restriction_low}}"
    %}

    {% big_value
        data="metric_calculation_total_sessions"
        value="sessions"
        comparison={
            compare_vs="target"
            target="sessions_last_year"
            text="{{$translations.vs_last_year}}"
            display_type="pct"
            delta=true
            pct_fmt="pct1"
        }
        title="{{$translations.metric_definitions.utilization.total_sessions.title}}"
        info="{{$translations.metric_definitions.utilization.total_sessions.description}}"
        info_link="#{{$translations.metric_definitions.utilization.total_sessions.info_link}}"
        info_link_title="{{ $translations.read_more}}"
    /%}


{% /if %}

{% else %}

    {% callout
        type="warning"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.total_sessions.title}} {{program_filter.label}} {{$translations.privacy_disclaimer_end}}
    {% /callout %}


{% /else %}
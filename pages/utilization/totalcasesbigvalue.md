---
name: total_cases_big_value
assetId: b20cdd29-8be2-4723-824b-bab6b1a778bd
type: partial
---

```sql metric_calculation_total_cases
with client_report_time as (
select 
    sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then cases end) as cases_all_current_year
    , sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then cases end) as cases_all_last_year

    , sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then cases_eap end) as cases_eap_current_year
    , sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then cases_eap end) as cases_eap_last_year

    , sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then cases_mental_health end) as cases_mental_health_current_year
    , sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then cases_mental_health end) as cases_mental_health_last_year

    , sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then cases_primary_care end) as cases_primary_care_current_year
    , sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then cases_primary_care end) as cases_primary_care_last_year

from client_reporting_monthly
where {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and date_month <= {{date_end.selected}}
    and date_month >= date_trunc('month',addMonths({{date_start.selected}}, -12))
)
, program_specific_values as (
select
cases_{{program_filter.literal}}_current_year
, case
    when cases_{{program_filter.literal}}_last_year < {{$privacy_restriction_low}} then null
    else cases_{{program_filter.literal}}_last_year
end as cases_{{program_filter.literal}}_last_year
from client_report_time
)

select
cases_{{program_filter.literal}}_current_year as cases
, cases_{{program_filter.literal}}_last_year as cases_last_year
from program_specific_values
```

{% if
    data="metric_calculation_total_cases"
    where="cases >= {{$privacy_restriction_low}}"
    %}

    {% big_value
        data="metric_calculation_total_cases"
        value="cases"
        comparison={
            compare_vs="target"
            target="cases_last_year"
            text="{{$translations.vs_last_year}}"
            display_type="pct"
            delta=true
            pct_fmt="pct1"
        }
        title="{{$translations.metric_definitions.utilization.total_cases.title}}"
        info="{{$translations.metric_definitions.utilization.total_cases.description}}"
        info_link="#{{$translations.metric_definitions.utilization.total_cases.info_link}}"
        info_link_title="{{ $translations.read_more}}"
    /%}


{% /if %}

{% else %}

    {% callout
        type="warning"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.total_cases.title}} {{program_filter.label}} {{$translations.privacy_disclaimer_end}}
    {% /callout %}


{% /else %}
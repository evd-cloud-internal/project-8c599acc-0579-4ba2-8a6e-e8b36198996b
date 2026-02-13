---
name: time_saved_avg
assetId: 42edb6a4-5042-4089-a6f0-2a4d0a7b2255
type: partial
---

```sql metric_calculation_avg_time_saved
with nps_patient_survey_additional_questions_program_mapping as (
select
    case
        when {{program_filter.selected}} = 'all' then 'all'
        else program
    end as program
    , *
from nps_patient_survey_additional_questions
where program is not null
and program not in ('wellness', 'mso')   
)
, metric_calc as (
select
round(avg(case when date_month between {{date_start.selected}} and {{date_end.selected}} then time_saved_hours end),1) as avg_time_saved

, round(avg(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then time_saved_hours end),1) as avg_time_saved_last_year

, sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} and time_saved_hours is not null then 1 end) as surveys_current_year

, sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) and time_saved_hours is not null then 1 end) as surveys_last_year

from nps_patient_survey_additional_questions_program_mapping
where date_month <= {{date_end.selected}}
and program = {{program_filter.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
and date_month < date_trunc('month',today())
)

select
avg_time_saved
, case
    when surveys_last_year < {{$privacy_restriction_low}} then null
    else avg_time_saved_last_year
end as avg_time_saved_last_year
, surveys_current_year
from metric_calc
```

{% if
    data="metric_calculation_avg_time_saved"
    condition="has_rows"
    where="surveys_current_year >= {{$privacy_restriction_low}}"
%}

    {% big_value
        data="metric_calculation_avg_time_saved"
        value="avg_time_saved"
        title="{{$translations.metric_definitions.employee_impact.time_saved.title}}"
        info="{{$translations.metric_definitions.employee_impact.time_saved.description}}"
        comparison={
            delta=true
            compare_vs="target"
            target="avg_time_saved_last_year"
            text="{{$translations.vs_last_year}}"
            pct_fmt="pct1"
            abs_fmt="num1"
        }
        info_link="#{{$translations.metric_definitions.employee_impact.time_saved.info_link}}"
        info_link_title="{{$translations.read_more}}"
    /%}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.time_saved.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}


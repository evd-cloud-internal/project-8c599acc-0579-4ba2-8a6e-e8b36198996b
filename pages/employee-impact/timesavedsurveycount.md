---
name: time_saved_survey_count
assetId: fdb48e49-75a2-4211-ab89-ec71d939072b
type: partial
---

```sql metric_calculation_time_saved_survey_count
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
sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} and time_saved_hours is not null then 1 end) as surveys_current_year

, sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) and time_saved_hours is not null then 1 end) as surveys_last_year

from nps_patient_survey_additional_questions_program_mapping
where date_month <= {{date_end.selected}}
and program = {{program_filter.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
and date_month < date_trunc('month',today())
)

select
surveys_current_year
, case
    when surveys_last_year < {{$privacy_restriction_low}} then null
    else surveys_last_year
end as surveys_last_year
from metric_calc
```

<!-- ```sql time_saved_survey_count_all_programs_selected
select
{{program_filter.selected}} as program_selection
```

{% if
    data="time_saved_survey_count_all_programs_selected"
    condition="has_rows"
    where="program_selection = 'all'"
    %}

    {% callout
        type="info"
        %}
        {{$translations.metric_definitions.employee_impact.time_saved_surveys.title}} {{$translations.all_program_restrictions}}
    {% /callout %}

{% /if %} -->

{% if
    data="metric_calculation_time_saved_survey_count"
    condition="has_rows"
    where="surveys_current_year >= {{$privacy_restriction_low}}"
    %}

    {% big_value
        data="metric_calculation_time_saved_survey_count"
        value="surveys_current_year"
        title="{{$translations.metric_definitions.employee_impact.time_saved_surveys.title}}"
        info="{{$translations.metric_definitions.employee_impact.time_saved_surveys.description}}"
        comparison={
            delta=true
            compare_vs="target"
            target="surveys_last_year"
            text="{{$translations.vs_last_year}}"
            pct_fmt="pct1"
            abs_fmt="num1"
        }
        info_link="#{{$translations.metric_definitions.employee_impact.time_saved_surveys.info_link}}"
        info_link_title="{{$translations.read_more}}"
    /%}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.time_saved_surveys.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}


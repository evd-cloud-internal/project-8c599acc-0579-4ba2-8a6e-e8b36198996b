---
name: time_saved_participation_rate
assetId: 45605c1b-7afc-47c8-a1af-65b5676f1495
type: partial
---

```sql metric_calculation_time_saved_participation_rate
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
, time_saved_users as (
select

'join field' as join_field

, count(distinct case when date_month between {{date_start.selected}} and {{date_end.selected}} and time_saved_hours is not null then user_id end) as time_saved_users_current_year

, count(distinct case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) and time_saved_hours is not null then user_id end) as time_saved_users_last_year

, sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} and time_saved_hours is not null then 1 end) as surveys_current_year

, sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) and time_saved_hours is not null then 1 end) as surveys_last_year

from nps_patient_survey_additional_questions_program_mapping
where date_month <= {{date_end.selected}}
and program = {{program_filter.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
and date_month < date_trunc('month',today())
)

, unique_active_users as (
    select 

    'join field' as join_field

    , count(distinct case when date_month between {{date_start.selected}} and {{date_end.selected}} then user_id end) as total_active_users_current_year

    , count(distinct case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then user_id end) as total_active_users_last_year

    from active_users
    where date_month <= {{date_end.selected}}
    and date_month >= date_trunc('month',addMonths({{date_start.selected}}, -12))
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and date_month < date_trunc('month',today())
    group by 1
)

, agg as (
select
    surveys_current_year
    , surveys_last_year

    , time_saved_users_current_year::float / total_active_users_current_year::float as participation_rate_current_year

    , time_saved_users_last_year::float / total_active_users_last_year::float as participation_rate_last_year

from unique_active_users
inner join time_saved_users
    on unique_active_users.join_field = time_saved_users.join_field

)

select
    surveys_current_year
    , participation_rate_current_year

    , case
        when surveys_last_year < {{$privacy_restriction_low}} then null
        else participation_rate_last_year
    end as participation_rate_last_year

from agg
```

{% if
    data="metric_calculation_time_saved_participation_rate"
    condition="has_rows"
    where="surveys_current_year >= {{$privacy_restriction_low}}"
    %}

    {% big_value
        data="metric_calculation_time_saved_participation_rate"
        value="participation_rate_current_year"
        title="{{$translations.metric_definitions.employee_impact.time_saved_participation_rate.title}}"
        fmt="pct1"
        info="{{$translations.metric_definitions.employee_impact.time_saved_participation_rate.title}}"
        comparison={
            delta=true
            compare_vs="target"
            target="participation_rate_last_year"
            text="{{$translations.vs_last_year}}"
            pct_fmt="pct1"
            abs_fmt="num1"
        }
        info_link="#{{$translations.metric_definitions.employee_impact.time_saved_participation_rate.info_link}}"
        info_link_title="{{$translations.read_more}}"
    /%}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.time_saved_participation_rate.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}


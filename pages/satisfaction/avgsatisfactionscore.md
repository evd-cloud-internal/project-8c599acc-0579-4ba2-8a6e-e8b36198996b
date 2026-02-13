---
name: avg_satisfaction_score
assetId: 5284f96b-9c12-446c-b0b5-33dd07c1dfd4
type: partial
---

```sql metric_calculation_avg_satisfaction
with nps_patient_survey_program_mapping as (
select
    case
        when {{program_filter.selected}} = 'all' then 'all'
        else program
    end as program
    , date_month, score
from nps_patient_survey
where program is not null
and program not in ('wellness', 'mso')
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)

, agg as (
select
avg(case when date_month between {{date_start.selected}} and {{date_end.selected}} then score end) as avg_score

, avg(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then score end) as avg_score_last_year

, sum(case when score is not null and date_month between {{date_start.selected}} and {{date_end.selected}} then 1 else 0 end) as surveys

, sum(case when score is not null and date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then 1 else 0 end) as surveys_last_year

from nps_patient_survey_program_mapping
where date_month <= {{date_end.selected}}
and program = {{program_filter.selected}}

)

select
avg_score
, case
    when surveys_last_year < {{$privacy_restriction_low}} then null
    else avg_score_last_year
end as avg_score_last_year
, surveys
from agg
```


{% if
    data="metric_calculation_avg_satisfaction"
    condition="has_rows"
    where="surveys >= {{$privacy_restriction_low}}"
    %}

    {% row
    card=true
    %}

        {% big_value
            data="metric_calculation_avg_satisfaction"
            value="avg_score"
            title="{{$translations.metric_definitions.satisfaction.avg_satisfaction_score.title}}"
            info="{{$translations.metric_definitions.satisfaction.avg_satisfaction_score.description}} "
            comparison={
                delta=true
                compare_vs="target"
                target="avg_score_last_year"
                text="{{$translations.vs_last_year}}"
                pct_fmt="pct1"
                abs_fmt="num2"
            }
            info_link="#{{$translations.metric_definitions.satisfaction.avg_satisfaction_score.info_link}}"
            info_link_title="{{ $translations.read_more}}"
        /%}

    {% /row %}

{% /if %}

{% else %}

    {% callout
        type="warning"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.satisfaction.avg_satisfaction_score.title}}{{$translations.privacy_disclaimer_end}}

    {% /callout %}

{% /else %}




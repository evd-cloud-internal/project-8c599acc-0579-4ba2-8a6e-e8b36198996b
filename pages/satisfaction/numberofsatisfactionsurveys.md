---
name: number_of_satisfaction_surveys
assetId: d688f6e2-3230-4909-8ff4-11571fa46d48
type: partial
---

```sql metric_calculation_survey_count
with nps_patient_survey_program_mapping as (
select
    case
        when {{program_filter.selected}} = 'all' then 'all'
        else program
    end as program
    , *
from nps_patient_survey
where program is not null
and program not in ('wellness', 'mso')   
)

, agg as (
select

sum(case when score is not null and date_month between {{date_start.selected}} and {{date_end.selected}} then 1 else 0 end) as surveys

, sum(case when score is not null and date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then 1 else 0 end) as surveys_last_year

from nps_patient_survey_program_mapping
where date_month <= {{date_end.selected}}
and date_month < date_trunc('month',today())
and program = {{program_filter.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)
select
surveys
, case
    when surveys_last_year < {{$privacy_restriction_low}} then null
    else surveys_last_year
end as surveys_last_year
from agg
```

{% if
    data="metric_calculation_survey_count"
    condition="has_rows"
    where="surveys >= {{$privacy_restriction_low}}"
    %}

    {% row
        card=true
        %}

        {% big_value
            data="metric_calculation_survey_count"
            value="surveys"
            title="{{$translations.metric_definitions.satisfaction.satisfaction_surveys.title}}"
            info="{{$translations.metric_definitions.satisfaction.satisfaction_surveys.description}}"
            comparison={
                delta=true
                target="surveys_last_year"
                compare_vs="target"
                text="{{$translations.vs_last_year}}"
                pct_fmt="pct1"
            }
            info_link="#{{$translations.metric_definitions.satisfaction.satisfaction_surveys.info_link}}"
            info_link_title="{{ $translations.read_more}}"
        /%}

    {% /row %}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.satisfaction.satisfaction_surveys.title}}{{$translations.privacy_disclaimer_end}}

    {% /callout %}

{% /else %}


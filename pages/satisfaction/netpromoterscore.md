---
name: net_promoter_score
assetId: 36b26f6f-2817-4a13-8f91-a081a109702e
type: partial
---

```sql metric_calculation_nps
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
100*(sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then promoters end) - sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then detractors end)) / sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then surveys end) as nps

, 100*(sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then promoters end) - sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then detractors end)) / sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then surveys end) as nps_last_year

, sum(case when date_month between {{date_start.selected}} and {{date_end.selected}} then surveys end) as surveys_count

, sum(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then surveys end) as surveys_last_year

from nps_patient_survey_program_mapping
where date_month < date_trunc('month',today())
    and date_month <= {{date_end.selected}}
    and program = {{program_filter.selected}}
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
)

select
nps
, case
    when surveys_last_year < {{$privacy_restriction_low}} then null
    else nps_last_year
end as nps_last_year
, surveys_count
from agg
```


{% if
    data="metric_calculation_nps"
    condition="has_rows"
    where="surveys_count >= {{$privacy_restriction_low}} "
    %}

    {% row
        card=true
        %}

        {% big_value
            data="metric_calculation_nps"
            value="nps"
            title="{{$translations.metric_definitions.satisfaction.nps.title}}"
            info="{{$translations.metric_definitions.satisfaction.nps.description}}"
            comparison={
                delta=true
                compare_vs="target"
                target="nps_last_year"
                text="{{$translations.vs_last_year}}"
                pct_fmt="pct1"
                abs_fmt="num1"
            }
            info_link="#{{$translations.metric_definitions.satisfaction.nps.info_link}}"
            info_link_title="{{ $translations.read_more}}"
        /%}

    {% /row %}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.satisfaction.nps.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}



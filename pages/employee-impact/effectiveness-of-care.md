---
name: effectiveness of care
assetId: c728ae4b-2c85-491b-8092-9782d1437c2b
type: partial
---

---
program: primary_care
---

```sql metric_calculation_effectiveness_of_care
with effectiveness_of_care as (
select episode_id
    , date_month_est
    , issue_type_reporting_en
    , case when symptoms_follow_up_answer = 'better' then 1 end as feeling_better 
    , episode_interval_id
from episodes_with_contracts
where date_month_est <= {{date_end.selected}}
    and date_month_est < date_trunc('month',today())
    and outcome = 'md_np_appointment'
    and is_symptoms_follow_up_answered
    and program = '{{ $program }}'
    and issue_type_reporting_en is not null
    and is_suitable_episode
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
)
, joined as (
select
    round(count(distinct case when feeling_better = 1 and
    date_month_est between {{date_start.selected}} and {{date_end.selected}}
    then episode_id end)::float 
    / count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then episode_id end),5)*100 as effectiveness_overall
    , round(count(distinct case when feeling_better = 1 and
    date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12))
    then episode_id end)::float 
    / count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then episode_id end),5)*100 as effectiveness_overall_last_year
    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then episode_id end) as number_cases
    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then episode_id end) as number_cases_last_year
from effectiveness_of_care
)

select
effectiveness_overall
, case
    when number_cases_last_year < {{$privacy_restriction_medium}} then null
    else effectiveness_overall_last_year
end as effectiveness_overall_last_year
, number_cases
, number_cases_last_year
from joined
```

{% stack card=true %}

    ### {{$translations.section_descriptions.employee_impact.physical_health.title}}
    <!-- Physical Health -->
    <!-- Indicates the percentage of cases where members reported feeling better after consulting for physical health symptoms. -->
    {{$translations.section_descriptions.employee_impact.physical_health.overview}}

    {% if
        data="metric_calculation_effectiveness_of_care"
        condition="has_rows"
        where="number_cases >= {{$privacy_restriction_medium}}"
        %}

        {% row card=true %}

            {% big_value
                data="metric_calculation_effectiveness_of_care"
                value="effectiveness_overall"
                title="{{$translations.metric_definitions.employee_impact.effectiveness_of_care.title}}"
                info="{{$translations.metric_definitions.employee_impact.effectiveness_of_care.description}}"
                comparison={
                    delta=true
                    compare_vs="target"
                    target="effectiveness_overall_last_year"
                    text="{{$translations.vs_last_year}}"
                    abs_fmt="num1"
                    pct_fmt="pct1"
                }
                info_link="#{{$translations.metric_definitions.employee_impact.effectiveness_of_care.info_link}}"
                info_link_title="{{$translations.read_more}}"
                
            /%}

        {% /row %}    

    {% /if %}

    {% else %}

        {% callout
        type="warning"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.effectiveness_of_care.title}}{{$translations.privacy_disclaimer_end}}
        {% /callout %}

    {% /else %}

{% /stack %} 


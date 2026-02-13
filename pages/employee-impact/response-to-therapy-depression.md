---
name: response to therapy depression
assetId: b2699835-0997-4b2a-8b8d-91ae0fe18b39
type: partial
---

```sql metric_desc_response_ther_depression
select
    value as description
from base_client_reporting_metric_descriptions
where metric_name = 'Time to Response to Therapy (Depression)'
    and language = 'EN'
```

```sql metric_calculation_response_ther_depression
with time_to_response_phq9 as (
    select 
        date_trunc('month', first_phq9_timestamp) as date_month 
        , episode_id
        , dateDiff('day', first_phq9_timestamp, completed_at) as time_to_response_to_therapy_phq9
        , completed_at as reached_response_to_therapy_phq9_at
    from  mh_scores_phq9_with_contracts
    where difference_from_first_score < -0.40
        and initial_phq9_score >= 10
        and date_trunc('month', first_phq9_timestamp) <= {{date_end.selected}}
        and date_trunc('month', first_phq9_timestamp) < date_trunc('month',today())
        and {{filter_value.filter}}
        and reporting_block_id = '{{ $reporting_block_id }}'
    qualify rank() over (partition by episode_id order by completed_at) = 1
)

select 
median(case when date_month between {{date_start.selected}} and {{date_end.selected}} then time_to_response_to_therapy_phq9 end) as time_to_response_to_therapy_days

, median(case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then time_to_response_to_therapy_phq9 end) as time_to_response_to_therapy_days_last_year

, count(distinct case when date_month between {{date_start.selected}} and {{date_end.selected}} then episode_id end) as number_cases

, count(distinct case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then episode_id end) as number_cases_last_year

from time_to_response_phq9
```

```sql restriction_challenges_response_ther_depression
select
time_to_response_to_therapy_days
, case
    when number_cases_last_year < {{$privacy_restriction_medium}} then null
    else time_to_response_to_therapy_days_last_year
end as time_to_response_to_therapy_days_last_year
, number_cases 
from {{metric_calculation_response_ther_depression}} as metric_calculation
```

{% if
    data="restriction_challenges_response_ther_depression"
    condition="has_rows"
    where="number_cases >= {{$privacy_restriction_medium}}"
    %}

    {% big_value
        data="restriction_challenges_response_ther_depression"
        value="time_to_response_to_therapy_days"
        title="{{$translations.metric_definitions.employee_impact.time_to_response_depression.title}}"
        info="{{$translations.metric_definitions.employee_impact.time_to_response_depression.description}}"
        comparison={
            delta=true
            compare_vs="target"
            target="time_to_response_to_therapy_days_last_year"
            text="{{$translations.vs_last_year}}"
            down_is_good=true
            abs_fmt="num1"
            pct_fmt="pct1"
        }
        info_link="#{{$translations.metric_definitions.employee_impact.time_to_response_depression.info_link}}"
        info_link_title="{{$translations.read_more}}"
    /%}

{% /if %}

{% else %}

    {% callout
        type="warning"
        %}

        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.time_to_response_depression.title}} {{$translations.privacy_disclaimer_end}}
        
    {% /callout %}

{% /else %}

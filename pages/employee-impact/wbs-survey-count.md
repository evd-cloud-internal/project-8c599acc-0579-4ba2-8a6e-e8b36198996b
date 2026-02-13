---
name: WBS survey count
assetId: ed3a30f1-0562-4eda-a76e-582dc8853a1a
type: partial
---

```sql metric_calculation_wellbeing_score_survey_count

with wbs_scores as (
SELECT qnaire_id as qnaire_tid
    , user_id
    , score
    , mood_score::integer as wbs_score_1
    , stress_score::integer as wbs_score_2
    , activeness_score::integer as wbs_score_3
    , sleep_score::integer as wbs_score_4
    , purpose_score::integer as wbs_score_5
    , wbs_total_score
    , wbs_completed_at_est
    , date_trunc('day', wbs_completed_at_est) as date_day_est
    , date_trunc('month', wbs_completed_at_est) as date_month_est
    , organization_id::integer as organization_id
    , reporting_group_id
FROM wbs_questionnaires_with_contracts
where date_month_est <= {{date_end.selected}}
and date_month_est < date_trunc('month',today())
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)

, agg as (
select


count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then qnaire_tid end) as total_qnaire_tid

, count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then qnaire_tid end) as total_qnaire_tid_last_year

from wbs_scores
)

select

case 
    when total_qnaire_tid_last_year < {{$privacy_restriction_low}} then null
    else total_qnaire_tid_last_year
end as total_qnaire_tid_last_year
, total_qnaire_tid

from agg
```

{% if
    data="metric_calculation_wellbeing_score_survey_count"
    condition="has_rows"
    where="total_qnaire_tid >= {{$privacy_restriction_low}}" 
    %}

    {% row card=true %}

        {% big_value
            data="metric_calculation_wellbeing_score_survey_count"
            value="total_qnaire_tid as survey_count"
            title="{{$translations.metric_definitions.employee_impact.wbs_surveys.title}}"
            info="{{$translations.metric_definitions.employee_impact.wbs_surveys.description}}"
            comparison={
                compare_vs="target"
                delta=true
                target="total_qnaire_tid_last_year"
                text="{{$translations.vs_last_year}}"
                abs_fmt="num1"
                pct_fmt="pct1"
            }
            info_link="#{{$translations.metric_definitions.employee_impact.wbs_surveys.info_link}}"
            info_link_title="{{$translations.read_more}}"
        /%}

    {% /row %}

{% /if %}

{% else %}

    {% callout
        type="warning"
        %}

        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.wbs_surveys.title}}{{$translations.privacy_disclaimer_end}}

    {% /callout %}

{% /else%}



---
name: WBS focus areas
assetId: 0962e492-8591-43f5-b6fc-e808928177e7
type: partial
---

```sql metric_calculation_wbs_focus
with wbs_scores as (
SELECT qnaire_id as qnaire_tid
    , user_id
    , score
    , mood_score
    , stress_score
    , activeness_score
    , sleep_score
    , purpose_score
    , wbs_total_score
    , wbs_completed_at_est
    , date_trunc('day', wbs_completed_at_est) as date_day_est
    , date_trunc('month', wbs_completed_at_est) as date_month_est
    , organization_id::integer as organization_id
    , reporting_group_id
FROM wbs_questionnaires_with_contracts
where date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and {{date_end.selected}}
and date_month_est < date_trunc('month',today())
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)

select
    '{{ $translations.metric_definitions.employee_impact.wbs_focus_area.mood}}' as category
    , avg(case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then mood_score end) as score
    , avg(case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then mood_score end) as score_last_year
    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then user_id end) as unique_users
    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then user_id end) as unique_users_last_year
from wbs_scores
group by 1
union all
select
    '{{ $translations.metric_definitions.employee_impact.wbs_focus_area.stress}}' as category
    , avg(case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then stress_score end) as score
    , avg(case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then stress_score end) as score_last_year
    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then user_id end) as unique_users
    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then user_id end) as unique_users_last_year
from wbs_scores
group by 1
union all
select
    '{{ $translations.metric_definitions.employee_impact.wbs_focus_area.activeness}}' as category
    , avg(case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then activeness_score end) as score
    , avg(case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then activeness_score end) as score_last_year
    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then user_id end) as unique_users
    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then user_id end) as unique_users_last_year
from wbs_scores
group by 1
union all
select
    '{{ $translations.metric_definitions.employee_impact.wbs_focus_area.sleep}}' as category
    , avg(case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then sleep_score end) as score
    , avg(case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then sleep_score end) as score_last_year
    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then user_id end) as unique_users
    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then user_id end) as unique_users_last_year
from wbs_scores
group by 1
union all
select
    '{{ $translations.metric_definitions.employee_impact.wbs_focus_area.purpose}}' as category
    , avg(case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then purpose_score end) as score
    , avg(case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then purpose_score end) as score_last_year
    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then user_id end) as unique_users
    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then user_id end) as unique_users_last_year
from wbs_scores
group by 1
```

```sql restrictions_wbs_focus
select category
    , case when unique_users < {{$privacy_restriction_low}} then null
    else score end as score
    , case when unique_users < {{$privacy_restriction_low}} then null
    else unique_users end as unique_users
    , case when unique_users_last_year < {{$privacy_restriction_low}} then null
    else score_last_year end as score_last_year
    , case when unique_users_last_year < {{$privacy_restriction_low}} then null
    else unique_users_last_year end as unique_users_last_year
    , sum(unique_users) over () as total_users
from {{metric_calculation_wbs_focus}}
```

    {% stack card=true %}

        ### {{$translations.section_descriptions.employee_impact.focus_areas.title}}
        <!-- Focus Areas -->

        <!-- Provides a breakdown of members' overall well-being across 5 dimensions (based on the World Health Organization’s WHO-5 Well-Being Index). -->
        {{$translations.section_descriptions.employee_impact.focus_areas.overview}}

{% if
    data="restrictions_wbs_focus"
    condition="has_rows"
    where="total_users is not null"
    %}

    {% row 
        align="center" %}

        {% table
            data="metric_calculation_wbs_focus"
            title="{{$translations.metric_definitions.employee_impact.wbs_focus_area.title}} "
            width=100
            info="{{$translations.metric_definitions.employee_impact.wbs_focus_area.description}}"
            info_link="#{{$translations.metric_definitions.employee_impact.wbs_focus_area.info_link}}"
            info_link_title="{{$translations.read_more}}"
            %}
            
            {% dimension
            value="category"
            title="{{$translations.metric_definitions.employee_impact.wbs_focus_area.focus_area}}"
            /%}

            {% dimension
            value="score"
            title="{{$translations.metric_definitions.employee_impact.wbs_focus_area.score}}"
            /%}

            {% measure
                value="score"
                comparison={
                    compare_vs="target"
                    target="score_last_year"
                }
                title=""
                viz="delta"
            /%}

        {% /table %}

    {% /row %}

{% /if %}

    {% else %}

            {% callout
                type="warning"
                %}
                
                {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.wbs_focus_area.title}}{{$translations.privacy_disclaimer_end}}
            
            {% /callout %}

    {% /else %}

{% /stack %}

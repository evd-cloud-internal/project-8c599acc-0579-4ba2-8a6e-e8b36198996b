---
name: WBS participation rate
assetId: 71bfcf08-472c-437e-b74a-c11d87e524fc
type: partial
---

```sql metric_calculation_wbs_participation_rate
with wbs as (
select
    wbs_questionnaires_with_contracts.qnaire_id as qnaire_tid
    , wbs_questionnaires_with_contracts.user_id
    , score
    , wbs_questionnaires_with_contracts.mood_score
    , wbs_questionnaires_with_contracts.stress_score
    , wbs_questionnaires_with_contracts.activeness_score
    , wbs_questionnaires_with_contracts.sleep_score
    , wbs_questionnaires_with_contracts.purpose_score
    , wbs_total_score
    , wbs_completed_at_est
    , date_trunc('day', wbs_completed_at_est) as date_day_est
    , date_trunc('month', wbs_completed_at_est) as date_month_est
    , wbs_questionnaires_with_contracts.reporting_group_id
from wbs_questionnaires_with_contracts
where date_month_est <= {{date_end.selected}}
and date_month_est >= date_trunc('month',addMonths({{date_start.selected}}, -12))
and date_month_est < date_trunc('month',today())
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
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

, wbs_users as (
select

    'join field' as join_field

    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then  user_id end) as total_wbs_users_current_year

    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then user_id end) as total_wbs_users_last_year

    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then qnaire_tid end) as total_qnaire_tid_current_year

    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then qnaire_tid end) as total_qnaire_tid_last_year

    from wbs
    where date_month_est <= {{date_end.selected}}
    and date_month_est >= date_trunc('month',addMonths({{date_start.selected}}, -12))
    group by 1
)

, agg as (
select
    total_qnaire_tid_current_year
    , total_qnaire_tid_last_year

    , total_wbs_users_current_year::float / total_active_users_current_year::float as participation_rate_current_year

    , total_wbs_users_last_year::float / total_active_users_last_year::float as participation_rate_last_year

from unique_active_users
inner join wbs_users
    on unique_active_users.join_field = wbs_users.join_field
)

select
total_qnaire_tid_current_year
, participation_rate_current_year
, case
    when total_qnaire_tid_last_year < {{$privacy_restriction_low}} then null
    else participation_rate_last_year
end as participation_rate_last_year
from agg
```

{% if
    data="metric_calculation_wbs_participation_rate"
    condition="has_rows"
    where="total_qnaire_tid_current_year >= {{$privacy_restriction_low}}"
    %}

    {% row card=true %}

        {% big_value
            data="metric_calculation_wbs_participation_rate"
            value="participation_rate_current_year"
            title="{{$translations.metric_definitions.employee_impact.wbs_participation_rate.title}} "
            info="{{$translations.metric_definitions.employee_impact.wbs_participation_rate.description}} "
            fmt="pct1"
            comparison={
                compare_vs="target"
                delta=true
                target="participation_rate_last_year"
                text="{{$translations.vs_last_year}}"
                abs_fmt="num1"
                pct_fmt="pct1"
            }
            info_link="#{{$translations.metric_definitions.employee_impact.wbs_participation_rate.info_link}}"
            info_link_title="{{$translations.read_more}}"
        /%}

    {% /row %}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.wbs_participation_rate.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}


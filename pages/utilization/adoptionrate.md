---
name: adoption_rate
assetId: 09731bae-969a-463f-9edf-fbfd3d19e8fd
type: partial
---

```sql metric_calculation_adoption_rate
with wellness_active as (
select toString(organization_id) as organization_id
    , count(distinct user_id) as unique_active_users
from wellness_activities_with_contracts_truncated
where date_month_est >= {{date_start.selected}}
    and date_month_est <= {{date_end.selected}}
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and date_month_est < date_trunc('month',today())
group by 1
)

, wellness_registered_employees as (
select organization_id
    , max(max_registered_employees) as max_registered_employees
from client_reporting_monthly
where avg_eligible_members_wellness > 0
    and date_month >= {{date_start.selected}}
    and date_month <= {{date_end.selected}}
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and date_month < date_trunc('month',today())
group by 1
)

, all_orgs as (
select wellness_registered_employees.organization_id
    , wellness_registered_employees.max_registered_employees
    , coalesce(wellness_active.unique_active_users,0) as unique_active_users
from wellness_registered_employees
left join wellness_active
on wellness_registered_employees.organization_id = wellness_active.organization_id
)

select sum(max_registered_employees) as registered_employees
    , sum(unique_active_users) as active_users
    , active_users::float/nullIf(registered_employees::float, 0) as adoption_rate
from all_orgs
```

```sql restriction_adoption_rate
select
registered_employees
, active_users
, adoption_rate
from {{metric_calculation_adoption_rate}} as metric_calculation
where active_users >= {{$privacy_restriction_low}}
```

{% if
    data="restriction_adoption_rate"
    condition="has_rows"
    %}

    <!-- {% row card=true %} -->
    {% big_value
        data="restriction_adoption_rate"
        value="adoption_rate"
        title="{{$translations.metric_definitions.utilization.adoption_rate.title}}"
        fmt="pct"
        info="{{$translations.metric_definitions.utilization.adoption_rate.description}}"
        info_link="#{{$translations.metric_definitions.utilization.adoption_rate.info_link}}"
        info_link_title="{{ $translations.read_more}}"
        /%}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.adoption_rate.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}



---
name: annualized_utilization_rate
assetId: 3ac22503-6654-4fd3-8a05-052d8c0d6464
type: partial
---

```sql metric_calculation_annualized_utilization_rate
with aliases as (
select
    date_month
    , organization_id
    , reporting_group_id
    , organization_id
    , sessions_primary_care as cases_primary_care
    , cases_mental_health
    , cases_eap
    , avg_eligible_members_primary_care
    , avg_eligible_members_mental_health
    , avg_eligible_members_eap
from client_reporting_monthly
where date_month < date_trunc('month',today())
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
)

, monthly as (
select
    date_month
    , sum(coalesce(aliases.cases_{{program_filter.literal}}, 0)) as cases
    , sum(coalesce(aliases.avg_eligible_members_{{program_filter.literal}}, 0)) as avg_eligible_members
    , min(accounts.number_of_months_since_renewal_{{program_filter.literal}}) as number_of_months_since_renewal
from aliases
left join organizations
    on aliases.organization_id = organizations.organization_id
left join accounts
    on organizations.account_id = accounts.account_id
where aliases.date_month >= accounts.latest_renewal_date_{{program_filter.literal}}
group by 1
)

, total as (
select
    sum(cases) as cases
    , avg(avg_eligible_members) as avg_eligible_members
    , avg(number_of_months_since_renewal) as avg_number_of_months_since_renewal
from monthly
)

select
    round((toFloat64(cases)/toFloat64(avg_eligible_members))/(avg_number_of_months_since_renewal - 1) * 12, 3) as annualized_utilization_rate
    , toFloat64(cases) as cases
    , toFloat64(avg_eligible_members) as avg_eligible_members
from total
```

```sql restrictions_annualized_utilization_rate
select
cases
, avg_eligible_members
, annualized_utilization_rate
from {{metric_calculation_annualized_utilization_rate}} as metric_calculation
where cases >= {{$privacy_restriction_low}}
and avg_eligible_members >= {{$privacy_restriction_high}}
```

```sql annualized_util_rate_all_programs_selected
select
{{program_filter.selected}} as program_selection
```


{% if
    data="annualized_util_rate_all_programs_selected"
    condition="has_rows"
    where="program_selection = 'all'"
    %}

    {% callout
        type="info"
        %}
        <!-- Annualized Utilization Rate is only available for **individual** programs -->
        {{$translations.metric_definitions.utilization.annualized_utilization_rate.title}} {{$translations.all_program_restrictions}}
    {% /callout %}

{% /if %}

{% else_if
    data="restrictions_annualized_utilization_rate"
    condition="has_rows"
    %}
<!-- pass privacy restrictions -->

    <!-- {% row card=true %} -->
    {% big_value
        data="restrictions_annualized_utilization_rate"
        value="annualized_utilization_rate"
        title="{{$translations.metric_definitions.utilization.annualized_utilization_rate.title}}"
        fmt="pct1"
        info="{{$translations.metric_definitions.utilization.annualized_utilization_rate.description}}"
        info_link="#{{$translations.metric_definitions.utilization.annualized_utilization_rate.info_link}}"
        info_link_title="{{ $translations.read_more}}"
    /%}

{% /else_if %}

{% else %}
    <!-- fail privacy restrictions -->
    {% callout
        type="warning"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.annualized_utilization_rate.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}


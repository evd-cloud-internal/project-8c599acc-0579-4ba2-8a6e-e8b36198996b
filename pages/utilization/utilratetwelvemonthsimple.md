---
name: util_rate_twelve_month_simple
assetId: 1385fea0-c2cb-48e2-b57c-7b1ddac0d2f6
type: partial
---

```sql metric_calculation_util_rate_12_month
with aliases as (
select
    date_month
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
    and date_month <= {{date_end.selected}}
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
)
, monthly as (
select
    date_month
    , sum(coalesce(aliases.cases_{{program_filter.literal}}, 0)) as cases
    , sum(coalesce(aliases.avg_eligible_members_{{program_filter.literal}}, 0)) as avg_eligible_members
    , row_number() over (order by date_month desc) as rank_month
from aliases
left join organizations 
    on aliases.organization_id = organizations.organization_id
left join accounts
    on organizations.account_id = accounts.account_id
group by 1
)

, total as ( 
select sum(cases) as cases
    , avg(avg_eligible_members) as avg_eligible_members
from monthly
where rank_month between 1 and 12
)

select
    round(toFloat64(cases)/ toFloat64(avg_eligible_members), 3) as util_rate_12_month
    , toFloat64(cases) as cases
    , toFloat64(avg_eligible_members) as avg_eligible_members
    , '{{program_filter.literal}}' as program
from total
```

```sql restrictions_util_rate_12_month
select
cases
, avg_eligible_members
from {{metric_calculation_util_rate_12_month}} as metric_calculation
where cases >= {{$privacy_restriction_low}}
and avg_eligible_members >= {{$privacy_restriction_high}}
```

```sql util_rate_12_month_all_programs_selected
select
{{program_filter.selected}} as program_selection
```


{% if
    data="util_rate_12_month_all_programs_selected"
    condition="has_rows"
    where="program_selection = 'all'"
    %}

    {% callout
        type="info"
        %}
        <!-- 12 Month Utilization Rate is only available for **individual** programs -->
        {{$translations.metric_definitions.utilization.twelve_month_utilization_rate.title}} {{$translations.all_program_restrictions}}
    {% /callout %}

{% /if %}

{% else_if
    data="restrictions_util_rate_12_month"
    condition="has_rows"
    %}
    <!-- restrictions passed ^^^ -->
    
    <!-- {% row card=true %} -->

        {% big_value
            data="metric_calculation_util_rate_12_month"
            value="util_rate_12_month"
            fmt="pct1"
            title="{{$translations.metric_definitions.utilization.twelve_month_utilization_rate.title}}"
            info="{{$translations.metric_definitions.utilization.twelve_month_utilization_rate.description}}"
            info_link="#{{$translations.metric_definitions.utilization.twelve_month_utilization_rate.info_link}}"
            info_link_title="{{ $translations.read_more}}"
        /%}

{% /else_if %}

{% else %}
    <!-- restrictions failed ^^^ -->
    {% callout
    type="warning"
    width=40
    %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.twelve_month_utilization_rate.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}

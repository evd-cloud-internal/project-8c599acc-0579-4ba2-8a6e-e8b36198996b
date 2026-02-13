---
name: utilization_rate_comparison
assetId: 9fdaf41b-a54b-479c-a7d3-eeaa5d0fbd61
type: partial
---

```sql metric_calculation_util_rate
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
    and ((date_month between {{date_start.selected}}
    and {{date_end.selected}})
    or (date_month between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12)))
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
)

, monthly as (
select
    date_month

    , sum(case when date_month between {{date_start.selected}}
    and {{date_end.selected}} then coalesce(aliases.cases_{{program_filter.literal}}, 0) end) as cases_current
    
    , case when 
    sum(case when date_month between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12) then coalesce(aliases.cases_{{program_filter.literal}}, 0) end) >= {{$privacy_restriction_low}}
    then
    sum(case when date_month between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12) then coalesce(aliases.cases_{{program_filter.literal}}, 0) end) 
    else null
    end as cases_previous

    , sum(case when date_month between {{date_start.selected}}
    and {{date_end.selected}} then coalesce(aliases.avg_eligible_members_{{program_filter.literal}}, 0) end) as avg_eligible_members_current

    , case when 
    sum(case when date_month between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12) then coalesce(aliases.avg_eligible_members_{{program_filter.literal}}, 0) end) >= {{$privacy_restriction_high}}
    then
    sum(case when date_month between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12) then coalesce(aliases.avg_eligible_members_{{program_filter.literal}}, 0) end) 
    else null
    end as avg_eligible_members_previous

from aliases
left join organizations 
    on aliases.organization_id = organizations.organization_id
left join accounts
    on organizations.account_id = accounts.account_id
group by 1
having sum(aliases.avg_eligible_members_{{program_filter.literal}}) > 0
)

, total as ( 
select
    sum(cases_current) as cases_current
    , avg(avg_eligible_members_current) as avg_eligible_members_current
    , sum(cases_previous) as cases_previous
    , avg(avg_eligible_members_previous) as avg_eligible_members_previous
from monthly
)

select
    round(toFloat64(cases_current)/ toFloat64(avg_eligible_members_current), 3) as util_rate_current
    , toFloat64(cases_current) as cases_current
    , toFloat64(avg_eligible_members_current) as avg_eligible_members_current

    , round(toFloat64(cases_previous)/ toFloat64(avg_eligible_members_previous), 3) as util_rate_previous
    , toFloat64(cases_previous) as cases_previous
    , toFloat64(avg_eligible_members_previous) as avg_eligible_members_previous
from total
```

```sql restrictions_util_rate
select
cases_current
, avg_eligible_members_current
, util_rate_current
, cases_previous
, avg_eligible_members_previous
, util_rate_previous
from {{metric_calculation_util_rate}} as metric_calculation
where cases_current >= {{$privacy_restriction_low}}
and avg_eligible_members_current >= {{$privacy_restriction_high}}
```

```sql util_rate_all_programs_selected
select
{{program_filter.selected}} as program_selection
```


{% if
    data="util_rate_all_programs_selected"
    condition="has_rows"
    where="program_selection = 'all'"
    %}

    {% callout
        type="info"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.utilization_rate.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /if %}

{% else_if
    data="restrictions_util_rate"
    condition="has_rows"
    %}
    <!-- pass privacy restrictions -->

    {% big_value
        data="restrictions_util_rate"
        value="util_rate_current"
        title="{{$translations.metric_definitions.utilization.utilization_rate.title}}"
        fmt="pct1"
        info="{{$translations.metric_definitions.utilization.utilization_rate.description}}"
        info_link="#{{$translations.metric_definitions.utilization.utilization_rate.info_link}}"
        info_link_title="{{ $translations.read_more}}"
        comparison={
            compare_vs="target"
            target="util_rate_previous"
            text="{{$translations.vs_last_year}}"
            pct_fmt="pct1"
        }
    /%}

{% /else_if %}

{% else%}
    <!-- fail privacy restrictions -->

    {% callout
        type="warning"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.utilization_rate.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}


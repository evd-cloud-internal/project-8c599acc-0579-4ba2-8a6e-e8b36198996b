---
name: consultation_per_case
assetId: 140410be-4d79-477c-9da4-95fc6c5a582e
type: partial
---

```sql metric_calculation_consultations_per_case

with episodes_with_contracts_program_mapping as (
select
    case
        when {{program_filter.selected}} = 'all' then 'all'
        else program
    end as program
    , *
from episodes_with_contracts
where program is not null
and program not in ('wellness', 'mso')   
)

select 
    avg(case when date_month_est between {{date_start.selected}}
    and {{date_end.selected}} then total_consults end) as average_consults_current

    , case when 
    count(distinct case when date_month_est between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12) then episode_id end) >= {{$privacy_restriction_low}}
    then
    avg(case when date_month_est between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12) then total_consults end) 
    else null
    end as average_consults_previous

    , count(distinct case when date_month_est between {{date_start.selected}}
    and {{date_end.selected}} then episode_id end) as distinct_episodes_current

    , case when 
    count(distinct case when date_month_est between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12) then episode_id end) >= {{$privacy_restriction_low}}
    then
    count(distinct case when date_month_est between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12) then episode_id end)
    else null
    end as distinct_episodes_previous

from episodes_with_contracts_program_mapping
where ((date_month_est >= {{date_start.selected}}
    and date_month_est <= {{date_end.selected}})
    or date_month_est between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12))
    and program = {{program_filter.selected}}
    and is_suitable_episode
    and date_month_est < date_trunc('month',today())
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
```

```sql restrictions_consultations_per_case
select
distinct_episodes_current
from {{metric_calculation_consultations_per_case}} as metric_calculation
where distinct_episodes_current >= {{$privacy_restriction_low}}
```

{% if
    data="restrictions_consultations_per_case"
    condition="has_rows"
    %}
    
    {% big_value
      data="metric_calculation_consultations_per_case"
      value="average_consults_current"
      title="{{$translations.metric_definitions.utilization.consultations_per_case.title}}"
      info="{{$translations.metric_definitions.utilization.consultations_per_case.description}}"
      fmt="num2"
      comparison={
        compare_vs="target"
        target="average_consults_previous"
        text="{{$translations.vs_last_year}}"
        pct_fmt="pct1"
      }
      info_link="#{{$translations.metric_definitions.utilization.consultations_per_case.info_link}}"
      info_link_title="{{ $translations.read_more}}"
    /%}


{% /if %}

{% else %}

{% callout
    type="warning"
    %}

    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.consultations_per_case.title}}{{$translations.privacy_disclaimer_end}}

{% /callout %}

{% /else %}


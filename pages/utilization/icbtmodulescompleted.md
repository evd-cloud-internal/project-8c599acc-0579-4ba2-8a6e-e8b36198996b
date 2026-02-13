---
name: icbt_modules_completed
assetId: b50db871-c530-45d9-8abd-5e3f065f8cd7
type: partial
---

```sql metric_calculation_icbt_modules_completed
with icbt_independent_activity as (
select
last_activity_at_month
, user_interval_id
, completion_rate
from icbt_independent_activity_with_contracts_truncated
where {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and ((last_activity_at_month between {{date_start.selected}}
    and {{date_end.selected}})
    or (last_activity_at_month between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12)))
)

select 
reporting_toolkit_{{ $translations.inline_query}} as reporting_toolkit
    -- Unique users for current period
    , count(distinct case 
        when last_activity_at_month between {{date_start.selected}} and {{date_end.selected}} 
        then user_interval_id 
    end) as unique_users_current,

    -- Unique users for previous period (only if privacy condition met)
    case 
        when count(distinct case 
            when last_activity_at_month between addMonths({{date_start.selected}}, -12) 
                 and addMonths({{date_end.selected}}, -12) 
            then user_interval_id 
        end) >= {{$privacy_restriction_low}}
        then count(distinct case 
            when last_activity_at_month between addMonths({{date_start.selected}}, -12) 
                 and addMonths({{date_end.selected}}, -12) 
            then user_interval_id 
        end)
        else null
    end as unique_users_previous,

    -- Average completion rate for current period
    avg(case 
        when last_activity_at_month between {{date_start.selected}} and {{date_end.selected}} 
        then completion_rate 
    end) as avg_completion_rate_current,

    -- Average completion rate for previous period (only if privacy condition met)
    case 
        when count(distinct case 
            when last_activity_at_month between addMonths({{date_start.selected}}, -12) 
                 and addMonths({{date_end.selected}}, -12) 
            then user_interval_id 
        end) >= {{$privacy_restriction_low}}
        then avg(case 
            when last_activity_at_month between addMonths({{date_start.selected}}, -12) 
                 and addMonths({{date_end.selected}}, -12) 
            then completion_rate 
        end)
        else null
    end as avg_completion_rate_previous
from icbt_page_view_with_contracts_truncated
inner join icbt_independent_activity using (user_interval_id)
group by 1
```


```sql restriction_icbt_modules_completed
select *
from {{metric_calculation_icbt_modules_completed}} as metric_calculation
where unique_users_current >= {{$privacy_restriction_low}}
```

{% stack card=true %}

    ### {{$translations.section_descriptions.utilization.icbt_modules.title}}
    <!-- iCBT Modules -->
    <!-- Measures participation and completion in digital therapy modules as an indicator of mental health self-management -->
    {{$translations.section_descriptions.utilization.icbt_modules.overview}}

{% if
    data="restriction_icbt_modules_completed"
    condition="has_rows"
%}

{% table
    data="restriction_icbt_modules_completed"
    order="avg_completion_rate_current desc"
    title="{{$translations.metric_definitions.utilization.icbt_modules.title}}"
    info="{{$translations.metric_definitions.utilization.icbt_modules.description}}"
    info_link="#{{$translations.metric_definitions.utilization.icbt_modules.info_link}}"
    info_link_title="{{ $translations.read_more}}"
%}
{% dimension
    value="reporting_toolkit"
    title="{{$translations.metric_definitions.utilization.icbt_modules.module}}"
/%}

{% dimension
    value="unique_users_current"
    title="{{$translations.metric_definitions.utilization.icbt_modules.unique_users}}"
    fmt="num"
/%}
{% measure
    value="unique_users_current"
    comparison={
        compare_vs="target"
        target="unique_users_previous"
        pct_fmt="pct1"
    }
    viz="delta"
    title=""
/%}

{% dimension
    value="avg_completion_rate_current"
    title="{{$translations.metric_definitions.utilization.icbt_modules.avg_completion_rate}}"
    fmt="pct1"
/%}

{% measure
    value="avg_completion_rate_current"
    fmt="pct1"
    comparison={
        compare_vs="target"
        target="avg_completion_rate_previous"
        pct_fmt="pct1"
    }
    viz="delta"
    title=""
/%}
{% /table %}

{% /if %}

{% else %}

{% callout
type="warning"
%}
{{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.icbt_modules.title}}{{$translations.privacy_disclaimer_end}}
{% /callout %}

{% /else %}

{% /stack %}
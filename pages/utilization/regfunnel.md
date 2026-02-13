---
name: reg_funnel
assetId: d79dc9a2-3856-46fb-80af-f8b888648887
type: partial
---

```sql metric_calculation_reg_funnel
with client_reporting_monthly_agg as (
    select
        max_eligible_members::float as eligible_members
        , case
            when {{member_type_filter.selected}} = 'All' then max_registered_employees::float + max_registered_family::float
            when {{member_type_filter.selected}} = 'Member' then max_registered_employees::float
            when {{member_type_filter.selected}} = 'Family' then max_registered_family::float
        end as registered_members
        , case
            when {{member_type_filter.selected}} = 'All' then max_activated_employees::float + max_activated_family::float
            when {{member_type_filter.selected}} = 'Member' then max_activated_employees::float
            when {{member_type_filter.selected}} = 'Family' then max_activated_family::float
        end as activated_members
    , *
    from client_reporting_monthly
)

, current_year_ranked_monthly as (
    select
        date_month
        , sum(eligible_members) as eligible_members
        , sum(registered_members) as registered_members
        , sum(activated_members) as activated_members
    from client_reporting_monthly_agg
    where date_month >= {{date_start.selected}}
    and date_month <= {{date_end.selected}}
    and date_month < date_trunc('month',today())
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    group by date_month
    order by date_month desc
    limit 1
)
, last_year_metrics as (
    select
        date_month
        , sum(eligible_members) as eligible_members_last_year
        , sum(registered_members) as registered_members_last_year
        , sum(activated_members) as activated_members_last_year
    from client_reporting_monthly_agg
    where date_month = date_trunc('month',addMonths({{date_end.selected}}, -12))
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    group by date_month
    order by date_month desc
    limit 1
)

select
date_month
, case
    when eligible_members < {{$privacy_restriction_high}} then null
    else eligible_members 
end as eligible_members

,case
    when eligible_members_last_year < {{$privacy_restriction_high}} then null
    else eligible_members_last_year 
end as eligible_members_last_year

,case
    when registered_members < {{$privacy_restriction_low}}  then null
    else registered_members 
end as registered_members

,case
    when registered_members_last_year < {{$privacy_restriction_low}} then null
    else registered_members_last_year 
end as registered_members_last_year

,case
    when activated_members < {{$privacy_restriction_low}} then null
    else activated_members 
end as activated_members

,case
    when activated_members_last_year < {{$privacy_restriction_low}} then null
    else activated_members_last_year 
end as activated_members_last_year
from current_year_ranked_monthly
left join last_year_metrics
    on date_trunc('month',addMonths(current_year_ranked_monthly.date_month, -12)) = last_year_metrics.date_month
```


```sql restrictions_master_table
select
'1. {{$translations.metric_definitions.utilization.activation_funnel.eligible}}' as member_status
, eligible_members as unique_count
, eligible_members_last_year as unique_count_last_year
, 0 as conversion_rate
, 0 as conversion_rate_last_year
, 1 as color_scaling
from {{metric_calculation_reg_funnel}}

union all 

select
'2. {{$translations.metric_definitions.utilization.activation_funnel.registered}}' as member_status
, registered_members as unique_count
, registered_members_last_year as unique_count_last_year
, coalesce(100*registered_members / eligible_members, 0) as conversion_rate
, coalesce(100*registered_members_last_year / eligible_members_last_year, 0) as conversion_rate_last_year
, 2 as color_scaling
from {{metric_calculation_reg_funnel}}

union all 

select
'3. {{$translations.metric_definitions.utilization.activation_funnel.activated}}' as member_status
, activated_members as unique_count
, activated_members_last_year as unique_count_last_year
, coalesce(100*activated_members / registered_members, 0) as conversion_rate
, coalesce(100*activated_members_last_year / registered_members_last_year, 0) as conversion_rate_last_year
, 3 as color_scaling
from {{metric_calculation_reg_funnel}}

```

{% stack card=true %}

    {% row card=false %}

        {% stack 
            card=false 
            width=70
            %}

        ### {{$translations.section_descriptions.utilization.activation_funnel_overview.title}}
        <!-- Activation Funnel - Overview -->
        
        <!-- Indicates how effectively eligible members progress from registration to first use. -->
        {{$translations.section_descriptions.utilization.activation_funnel_overview.overview}}
        
        {% /stack %}

        {% partial file="filters/membertypefilters"/%}

    {% /row %}

    {% if
        data="restrictions_master_table"
        condition="has_rows"
        where="unique_count is not null"
        %}

        {% funnel_chart
            data="restrictions_master_table"
            category="member_status"
            value="unique_count"
            align="center"
            order="member_status asc"
            
            gap=1
            where="unique_count is not null"
            title="{{$translations.metric_definitions.utilization.activation_funnel.title}}"
            info="{{$translations.metric_definitions.utilization.activation_funnel.description}}"
            info_link="#{{$translations.metric_definitions.utilization.activation_funnel.info_link}}"
            info_link_title="{{ $translations.read_more}}"
            label_position="outside"
            value_fmt="num"
        /%}

        {% table
            data="restrictions_master_table"
            where="unique_count is not null"
            order="member_status"
            %}

            {% dimension
            value="member_status"
            title="{{$translations.metric_definitions.utilization.activation_funnel.member_status}}"
            info="{{$translations.metric_definitions.utilization.activation_funnel.member_status_description}}"

            /%}

            {% dimension
            value="unique_count"
            title="{{$translations.metric_definitions.utilization.activation_funnel.unique_count}}"
            info="{{$translations.metric_definitions.utilization.activation_funnel.unique_count_description}}"
            fmt="num0"
            /%}

            {% measure
                value="unique_count"
                comparison={
                    compare_vs="target"
                    target="unique_count_last_year"
                    pct_fmt="pct1"
                }
                title=""
                viz="delta"
            /%}

            {% measure
                value="conversion_rate"
                viz="color"
                title="{{$translations.metric_definitions.utilization.activation_funnel.conversion_rate}} (%)"
                info="{{$translations.metric_definitions.utilization.activation_funnel.conversion_rate_description}}"
                color_options={
                    scale_column="color_scaling"
                }
            /%}

            {% measure
                value="conversion_rate"
                fmt="num1"
                comparison={
                    compare_vs="target"
                    target="conversion_rate_last_year"
                    pct_fmt="pct2"
                }
                title=""
                viz="delta"
            /%}

        {% /table %}

    {% /if %}

    {% else_if
        data="restrictions_master_table"
        condition="no_rows"
        where="unique_count is not null"
        %}

            {% callout
            type="warning"
            %}
            {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.activation_funnel.title}}{{$translations.privacy_disclaimer_end}}
            {% /callout %}

    {% /else_if %}

{% /stack %}


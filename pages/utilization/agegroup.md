---
name: age_group
assetId: c8499246-22c7-43a3-ad3f-590a4878f812
type: partial
---

```sql metric_calculation_age_group
with demographics as (
select 
    demographic_metrics_by_organization_monthly.date_month as date_month
    , demographic_metrics_by_organization_monthly.account_name
    , demographic_metrics_by_organization_monthly.account_id
    , demographic_metrics_by_organization_monthly.organization_name
    , demographic_metrics_by_organization_monthly.organization_id::integer as organization_id
    , demographic_metrics_by_organization_monthly.reporting_group_id
    , client_reporting_monthly.max_eligible_members
    , type
    , metric_name as metric_name_en
    , metric_name_fr
    , cast(metric_value as decimal(10,6)) as metric_value
from demographic_metrics_by_organization_monthly
left join client_reporting_monthly
    on demographic_metrics_by_organization_monthly.date_month = client_reporting_monthly.date_month
        and demographic_metrics_by_organization_monthly.reporting_group_id = client_reporting_monthly.reporting_group_id
where demographic_metrics_by_organization_monthly.date_month <> date_trunc('month',today())
and demographic_metrics_by_organization_monthly.date_month >= {{date_start.selected}}
and demographic_metrics_by_organization_monthly.date_month <= {{date_end.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)

, tmp as (select
metric_name_{{$translations.inline_query}} as metric_name
, date_month
, SUM(metric_value) / SUM(SUM(metric_value)) OVER () AS percentage
, SUM(metric_value) as bucket_count
, SUM(SUM(max_eligible_members)) OVER () as max_elig
, rank() over (order by date_month desc) as rank
from demographics
where type = 'generation'
group by 1,2
)

select 
case
    when metric_name = '{{$translations.metric_definitions.utilization.age_group.gen_alpha}}' then '{{$translations.metric_definitions.utilization.age_group.gen_alpha}} (0 - 12)'
    when metric_name = '{{$translations.metric_definitions.utilization.age_group.gen_z}}' then '{{$translations.metric_definitions.utilization.age_group.gen_z}} (12-28)'
    when metric_name = '{{$translations.metric_definitions.utilization.age_group.millenial}}' then '{{$translations.metric_definitions.utilization.age_group.millenial}} (28-44)'
    when metric_name = '{{$translations.metric_definitions.utilization.age_group.gen_x}}' then '{{$translations.metric_definitions.utilization.age_group.gen_x}} (44-59)'
    when metric_name = '{{$translations.metric_definitions.utilization.age_group.baby_boomer}}' then '{{$translations.metric_definitions.utilization.age_group.baby_boomer}} (59-79)'
    when metric_name = '{{$translations.metric_definitions.utilization.age_group.interwar}}' then '{{$translations.metric_definitions.utilization.age_group.interwar}} (79-97)' 
    when metric_name = '{{$translations.metric_definitions.utilization.age_group.greatest}}' then '{{$translations.metric_definitions.utilization.age_group.greatest}} (97+)'
    when metric_name = 'N/A' then 'N/A'
end as metric_name
, percentage
, bucket_count
, max_elig
from tmp
where rank = 1
```
```sql restrictions_age_group
with bucket_check as (
select
metric_name
, bucket_count
, max_elig
, percentage
from {{metric_calculation_age_group}} as metric_calculation
where bucket_count >= {{$privacy_restriction_demographics_bucket}}
)

select
metric_name
, round(percentage / sum(percentage) over (),2) as percent_masked
from bucket_check
where max_elig >= {{$privacy_restriction_demographics_members}}
```


{% if
    data="restrictions_age_group"
    condition="has_rows"
    %}

    {% bar_chart
        data="restrictions_age_group"
        x="metric_name"
        y="percent_masked as _"
        title="{{$translations.metric_definitions.utilization.age_group.title}}"
        chart_options={
            top_padding=5
        }
        data_labels={
            position="above"

        }
        y_fmt="pct0"
        x_axis_options={
            gridlines=false
            title=""
        }
        y_axis_options={
            gridlines=false
            labels=true

        }
        x_sort=["{{$translations.metric_definitions.utilization.age_group.gen_alpha}} (0 - 12)", "{{$translations.metric_definitions.utilization.age_group.gen_z}} (12-28)", "{{$translations.metric_definitions.utilization.age_group.millenial}} (28-44)", "{{$translations.metric_definitions.utilization.age_group.gen_x}} (44-59)", "{{$translations.metric_definitions.utilization.age_group.baby_boomer}} (59-79)", "{{$translations.metric_definitions.utilization.age_group.interwar}} (79-97)", "{{$translations.metric_definitions.utilization.age_group.greatest}} (97+)"]
        where="metric_name <> 'N/A'"
        order="percent_masked asc"
        qualify="percent_masked > 0"
        x_fmt="pct1"
        info="{{$translations.metric_definitions.utilization.age_group.description}}"
        info_link="#{{$translations.metric_definitions.utilization.age_group.info_link}}"
        info_link_title="{{ $translations.read_more}}"
    /%}

{% /if %}

{% else %}

    {% callout
        type="warning"
        %}
        
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.age_group.title}}{{$translations.privacy_disclaimer_end}}

    {% /callout %}

{% /else %}



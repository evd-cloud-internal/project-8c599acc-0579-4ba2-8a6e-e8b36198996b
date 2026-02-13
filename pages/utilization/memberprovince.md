---
name: member_province
assetId: efdd74b6-2c32-435b-8663-2a05f5837f9a
type: partial
---

```sql metric_calculation_member_province
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
and date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)

, tmp as (
select
    metric_name_{{ $translations.inline_query}} as metric_name
    , date_month
    , SUM(metric_value) / SUM(SUM(metric_value)) OVER () AS percentage
    , SUM(metric_value) as bucket_count
    , SUM(SUM(max_eligible_members)) OVER () as max_elig
    , rank() over (order by date_month desc) as rank
from demographics
where type = 'member_province'
group by 1,2
)

select metric_name
, percentage
, bucket_count
, max_elig
from tmp
where rank = 1
```


```sql restrictions_member_province
with bucket_check as (
select
metric_name
, bucket_count
, max_elig
, percentage
from {{metric_calculation_member_province}} as metric_calculation
where bucket_count >= {{$privacy_restriction_demographics_bucket}}
)

select
metric_name
, round(percentage / sum(percentage) over (),2) as percent_masked
from bucket_check
where max_elig >= {{$privacy_restriction_demographics_members}}
```


{% if
    data="restrictions_member_province"
    condition="has_rows"
    %}

    {% bar_chart
        data="restrictions_member_province"
        x="metric_name"
        y="percent_masked as _"
        data_labels={
            position="above"
        }
        y_fmt="pct0"
        info="{{$translations.metric_definitions.utilization.member_province.description}}"
        info_link="#{{$translations.metric_definitions.utilization.member_province.info_link}}"
        info_link_title="{{ $translations.read_more}}"
        title="{{$translations.metric_definitions.utilization.member_province.title}} "
        order="percent_masked desc"
        qualify="percent_masked > 0"
        x_fmt="pct1"
        x_axis_options={
            gridlines=false
            title=""
        }
        y_axis_options={
            gridlines=false
            title=""
        }
        chart_options={
            top_padding=5
        }
    /%}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.member_province.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else %}



---
name: OLD_member_province_comparison
assetId: a39330f3-8134-44c7-8e62-3237ed078a25
type: partial
---

---
privacy_restriction_value_bucket: 20
privacy_restriction_value_elig: 100
reporting_block_id: dia
---
# WIP, table comparison not used for now, as team agrees
{% partial
    file="combinefilters"
/%}
```sql metric_desc_member_province
select
value as description 
from base_client_reporting_metric_descriptions 
where metric_name = 'Member Province'
and language = 'EN'
```

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
    , metric_name
    , metric_name_fr
    , cast(metric_value as decimal(10,2)) as metric_value

from demographic_metrics_by_organization_monthly
left join client_reporting_monthly
    on demographic_metrics_by_organization_monthly.date_month = client_reporting_monthly.date_month
        and demographic_metrics_by_organization_monthly.reporting_group_id = client_reporting_monthly.reporting_group_id
where demographic_metrics_by_organization_monthly.date_month <> date_trunc('month',today())
and ((date_month between {{date_start.selected}} and {{date_end.selected}}) )
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)

, tmp as (select
metric_name
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
{% table
    data="metric_calculation_member_province"
%}
{% /table %}
```sql restrictions_member_province
with bucket_check as (
select
metric_name
, bucket_count
, max_elig
, percentage
from {{metric_calculation_member_province}} as metric_calculation
where bucket_count >= {{$privacy_restriction_value_bucket}}
)

select
metric_name
, round(percentage / sum(percentage) over (),2) as percent_masked
from bucket_check
where max_elig >= {{$privacy_restriction_value_elig}}
```


{% if
    data="restrictions_member_province"
    condition="has_rows"
%}

{% row
card=true 
%}

{% bar_chart
    data="restrictions_member_province"
    x="percent_masked as _ "
    y="metric_name"
    info="The percent distribution of self-declared provinces where activated members are situated in, provided upon initial sign up or later updated in the member app or website. If this field is not available, the members location will be taken from eligibility records."
    title="Member Province"
    order="percent_masked asc"
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
    
/%}


{% /row %}

{% /if %}

{% else %}

{% callout
type="warning"
%}
Metric value(s) cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else %}



---
name: OLD_age_group_comparison
assetId: e59ecced-cf6e-4afa-80b4-0d3ba48ef3de
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
    , metric_name
    , metric_name_fr
    , cast(metric_value as decimal(10,2)) as metric_value
from demographic_metrics_by_organization_monthly
left join client_reporting_monthly
    on demographic_metrics_by_organization_monthly.date_month = client_reporting_monthly.date_month
        and demographic_metrics_by_organization_monthly.reporting_group_id = client_reporting_monthly.reporting_group_id
where demographic_metrics_by_organization_monthly.date_month <> date_trunc('month',today())
and ((date_month between {{date_start.selected}}
and {{date_end.selected}}) or (date_month between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12)))
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)

, MonthFilter AS (
    SELECT
        MAX(date_month) as latest_month,
        addMonths(MAX(date_month), -12) as previous_year_month
    FROM demographics
)

, tmp as (select
metric_name
, date_month

, SUM(case when date_month between {{date_start.selected}} and {{date_end.selected}} then metric_value end) / SUM(SUM(case when date_month between {{date_start.selected}} and {{date_end.selected}} then metric_value end)) OVER () AS percentage_current

, 
case when 
SUM(case when date_month between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then metric_value end) >= {{$privacy_restriction_value_bucket}}
then
SUM(case when date_month between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then metric_value end) / SUM(SUM(case when date_month between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then metric_value end)) OVER () 
else null end AS percentage_previous

, SUM(case when date_month between {{date_start.selected}} and {{date_end.selected}} then metric_value end) as bucket_count_current

, SUM(case when date_month between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then metric_value end) as bucket_count_previous

, SUM(SUM(case when date_month between {{date_start.selected}} and {{date_end.selected}} then  max_eligible_members end)) OVER () as max_elig_current

, SUM(SUM(case when date_month between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then  max_eligible_members end)) OVER () as max_elig_previous

from demographics
inner join MonthFilter 
ON demographics.date_month = MonthFilter.latest_month
    OR demographics.date_month = MonthFilter.previous_year_month
where type = 'generation'
group by 1,2
)

select 
case
    when metric_name = 'Generation Alpha' then 'Generation Alpha (0 to 12)'
    when metric_name = 'Generation Z' then 'Generation Z (12-28)'
    when metric_name = 'Millennial' then 'Millenial (28-44)'
    when metric_name = 'Generation X' then 'Generation X (44-59)'
    when metric_name = 'Baby Boomer' then 'Baby Boomer (59-79)'
    when metric_name = 'Interwar Generation' then 'Interwar Generation (79-97)' 
    when metric_name = 'Greatest Generation' then 'Greatest Generation (97+)'
    when metric_name = 'N/A' then 'N/A'
end as metric_name
, percentage_current
, percentage_previous
, bucket_count_current
, bucket_count_previous
, max_elig_current
, max_elig_previous
from tmp
order by metric_name
```
{% table
    data="metric_calculation_age_group"
%}
{% /table %}
```sql restrictions_age_group
with bucket_check as (
select
metric_name
, bucket_count
, max_elig
, percentage_current
from {{metric_calculation_age_group}} as metric_calculation
where bucket_count >= {{$privacy_restriction_value_bucket}}
)

select
metric_name
, round(percentage_current / sum(percentage_current) over (),2) as percent_masked
from bucket_check
where max_elig >= {{$privacy_restriction_value_elig}}
```


{% if
    data="restrictions_age_group"
    condition="has_rows"
%}

    {% row
    card=true
    %}

    {% bar_chart
        data="restrictions_age_group"
        x="percent_masked as _"
        y="metric_name"
        title="Age groups"
        x_axis_options={
            gridlines=false
            title=""
        }
        y_axis_options={
            gridlines=false
        }
        order="percent_masked asc"
        qualify="percent_masked > 0"
        x_fmt="pct1"
        info="The percent distribution of ages of activated members based on self-declared date of birth grouped into Canadian government defined generations"
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



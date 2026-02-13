---
name: OLD_total_cases_this_quarter
assetId: 623c4fef-fbc5-4eb0-b907-85180cedc2be
type: partial
---

---
program: primary_care
---
```sql metric_desc_total_cases_this_q
select
value as description
from base_client_reporting_metric_descriptions
where language = 'EN'
and metric_name = 'Total Cases (This Quarter)'
```

```sql metric_calculation_total_cases_this_q
select
sum(case when date_trunc('quarter',date_month) = date_trunc('quarter',today()- interval '1 quarter') then cases_{{ $program }} else 0 end) as cases_last_q
, sum(case when date_trunc('quarter',date_month) = date_trunc('quarter',today()) then cases_{{ $program }} else 0 end) as cases_this_q
from client_reporting_monthly
where date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and date_month < date_trunc('month',today())
and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
```

```sql restriction_total_cases_this_q
with privacy_restriction_value as (
select
privacy_restriction_metric_value as value
from base_client_reporting_metric_privacy_restrictions
where privacy_restriction_metric_name = 'cases'
)

select
cases_this_q
from {{metric_calculation_total_cases_this_q}} as metric_calculation
cross join privacy_restriction_value
where cases_this_q >= privacy_restriction_value.value
```

```sql restriction_total_cases_last_q
with privacy_restriction_value as (
select
privacy_restriction_metric_value as value
from base_client_reporting_metric_privacy_restrictions
where privacy_restriction_metric_name = 'cases'
)

select
cases_last_q
from {{metric_calculation_total_cases_this_q}} as metric_calculation
cross join privacy_restriction_value
where cases_last_q >= privacy_restriction_value.value 
```

{% row card=true %}

{% if
    data="restriction_total_cases_this_q"
    condition="has_rows"
%}

{% row card=true %}

    {% big_value
        data="metric_calculation_total_cases_this_q"
        value="cases_this_q"
        title="Total Cases (this quarter)"
    /%}
    {% modal
        title="Info"
        icon_only=true
        variant="ghost"
        icon="info"
    %}
    {% table
        data="metric_desc_total_cases_this_q"
        dimensions=["description"]

    /%}
    {% /modal %}

{% /row %}

{% /if %}

{% else_if
    data="restriction_total_cases_this_q"
    condition="no_rows"
%}

{% callout
type="warning"
%}
Metric value(s) cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else_if %}

{% if
    data="restriction_total_cases_last_q"
    condition="has_rows"
%}

{% row card=true %}

    {% big_value
        data="metric_calculation_total_cases_this_q"
        value="cases_last_q"
        title="Total Cases (last quarter)"
    /%}
    {% modal
        title="Info"
        icon_only=true
        variant="ghost"
        icon="info"
    %}
    {% table
        data="metric_desc_total_cases_this_q"
        dimensions=["description"]

    /%}
    {% /modal %}

{% /row %}

{% /if %}

{% else_if
    data="restriction_total_cases_last_q"
    condition="no_rows"
%}

{% callout
type="warning"
%}
Total Cases cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else_if %}



{% /row %}

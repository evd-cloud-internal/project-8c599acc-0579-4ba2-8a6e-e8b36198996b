---
name: OLD_total_cases
assetId: cb9fbf88-51c7-4c4e-932f-62aeaefe5a0f
type: partial
---

```sql metric_desc_total_cases
select
value as description
from base_client_reporting_metric_descriptions
where language = 'EN'
and metric_name = 'Total Cases'
```

```sql metric_calculation_total_cases
select
sum(toFloat64(cases_primary_care)) as cases_primary_care
, sum(toFloat64(cases_mental_health)) as cases_mental_health
, sum(toFloat64(cases_eap)) as cases_eap
from client_reporting_monthly
where date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and date_month < date_trunc('month',today())
and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
```

```sql restriction_total_cases_PC
with privacy_restriction_value as (
select
privacy_restriction_metric_value as value
from base_client_reporting_metric_privacy_restrictions
where privacy_restriction_metric_name = 'cases'
)

select
cases_primary_care
from {{metric_calculation_total_cases}} as metric_calculation
cross join privacy_restriction_value
where cases_primary_care >= privacy_restriction_value.value
```

```sql restriction_total_cases_mh
with privacy_restriction_value as (
select
privacy_restriction_metric_value as value
from base_client_reporting_metric_privacy_restrictions
where privacy_restriction_metric_name = 'cases'
)

select
cases_mental_health
from {{metric_calculation_total_cases}} as metric_calculation
cross join privacy_restriction_value
where cases_mental_health >= privacy_restriction_value.value
```

```sql restriction_total_cases_eap
with privacy_restriction_value as (
select
privacy_restriction_metric_value as value
from base_client_reporting_metric_privacy_restrictions
where privacy_restriction_metric_name = 'cases'
)

select
cases_eap
from {{metric_calculation_total_cases}} as metric_calculation
cross join privacy_restriction_value
where cases_eap >= privacy_restriction_value.value
```

{% row card=true %}

{% if
    data="restriction_total_cases_PC"
    condition="has_rows"
%}

{% row %}

    {% big_value
        data="metric_calculation_total_cases"
        value="cases_primary_care"
        title="Total Cases - Primary Care"
    /%}

    {% modal
        title="Info"
        icon_only=true
        variant="ghost"
        icon="info"
    %}
    {% table
        data="metric_desc_total_cases"
        dimensions=["description"]
    /%}
    {% /modal %}

{% /row %}



{% /if %}

{% else_if
    data="restriction_total_cases_PC"
    condition="no_rows"
%}

{% callout
type="warning"
%}
Total Cases cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else_if %}

{% if
    data="restriction_total_cases_mh"
    condition="has_rows"
%}

{% row %}

    {% big_value
        data="metric_calculation_total_cases"
        value="cases_mental_health"
        title="Total Cases - Mental Health"
    /%}

    {% modal
        title="Info"
        icon_only=true
        variant="ghost"
        icon="info"
    %}
    {% table
        data="metric_desc_total_cases"
        dimensions=["description"]
    /%}
    {% /modal %}

{% /row %}



{% /if %}

{% else_if
    data="restriction_total_cases_mh"
    condition="no_rows"
%}

{% callout
type="warning"
%}
Total Cases cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else_if %}

{% if
    data="restriction_total_cases_eap"
    condition="has_rows"
%}

{% row %}

    {% big_value
        data="metric_calculation_total_cases"
        value="cases_eap"
        title="Total Cases - EAP"
    /%}


    {% modal
        title="Info"
        icon_only=true
        variant="ghost"
        icon="info"
    %}
    {% table
        data="metric_desc_total_cases"
        dimensions=["description"]
    /%}
    {% /modal %}

{% /row %}



{% /if %}

{% else_if
    data="restriction_total_cases_eap"
    condition="no_rows"
%}

{% callout
type="warning"
%}
Total Cases cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else_if %}

{% /row %}

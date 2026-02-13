---
name: OLD_eligible_members
assetId: 39561d03-1bdc-455f-900e-fb3da93bf22b
type: partial
---

```sql metric_desc_elig_member
select
    value as description
from base_client_reporting_metric_descriptions
where metric_name = 'Number of Eligible Members'
    and language = 'EN'
```

```sql metric_calculation_elig_member
with ranked_monthly as (
    select
        date_month
        , sum(max_eligible_members) as eligible_members
        , rank() over (order by date_month desc) as rank
    from client_reporting_monthly
    inner join {{filtered_table}} as filtered_table
        on client_reporting_monthly.reporting_group_id = filtered_table.reporting_group_id
    where date_month >= {{date_start.selected}}
    and date_month <= {{date_end.selected}}
    and date_month < date_trunc('month',today())
    group by date_month
)
select
    date_month,
    eligible_members
from ranked_monthly
where rank = 1
```

```sql restrictions_elig_member
with privacy_restriction_value as (
select
privacy_restriction_metric_value as value
from base_client_reporting_metric_privacy_restrictions
where privacy_restriction_metric_name = 'eligible_members'
)

select
eligible_members
from {{metric_calculation_elig_member}} as metric_calculation
cross join privacy_restriction_value
where eligible_members > privacy_restriction_value.value
```

{% if
    data="restrictions_elig_member"
    condition="has_rows"
%}

    {% big_value
        data="metric_calculation_elig_member"
        value="eligible_members"
        title="Total Eligible Members"
        info="The number of employees eligible for Dialogue, as of the last day of the latest month."
    /%}

{% /if %}

{% else_if
    data="restrictions_elig_member"
    condition="no_rows"
%}

{% callout
type="warning"
%}
Metric value(s) cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else_if %}





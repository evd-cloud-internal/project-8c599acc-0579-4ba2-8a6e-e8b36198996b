---
name: OLD_total_active_users
assetId: 015eee8f-c883-41d8-b8d7-8b6efe41f149
type: partial
---

```sql metric_desc_total_daus
select
value as description
from base_client_reporting_metric_descriptions
where language = 'EN'
and metric_name = 'Total Sessions'
```

```sql metric_calculation_total_daus
select
coalesce(sum(daus),0) as daily_active_users
from client_reporting_monthly
where date_month < date_trunc('month',today())
and date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
```

```sql restrictions_total_daus
with privacy_restriction_value as (
select
privacy_restriction_metric_value as value
from base_client_reporting_metric_privacy_restrictions
where privacy_restriction_metric_name = 'cases'
)

select
daily_active_users
from {{metric_calculation_total_daus}} as metric_calculation
cross join privacy_restriction_value
where daily_active_users > privacy_restriction_value.value

```


{% if
    data="restrictions_total_daus"
    condition="has_rows"
    %}

    {% big_value
        data="metric_calculation_total_daus"
        value="daily_active_users"
        title="Total Daily Active Users"
        info="The number of daily interactions with the care team, including consultations and chats."
    /%}

{% /if %}

{% else_if
    data="restrictions_total_daus"
    condition="no_rows"
    %}

    {% callout
    type="warning"
    %}
    Total Daily Active Users cannot be displayed because it is below privacy threshold
    {% /callout %}

{% /else_if %}


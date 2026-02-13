---
name: OLD_total_active_users_time_benchmark
assetId: d6519290-fbfb-4c86-b393-7f25d0d95eef
type: partial
---

---
privacy_restriction_value_dau: 5
---

```sql metric_desc_total_daus
select
value as description
from base_client_reporting_metric_descriptions
where language = 'EN'
and metric_name = 'Total Sessions'
```

```sql metric_calculation_total_daus
WITH
    toDate({{date_start.selected}}) AS start_date,
    toDate({{date_end.selected}}) AS end_date
select
coalesce(sum(daus),0) as daily_active_users
, offset
from client_reporting_monthly
ARRAY JOIN range(0, 2) AS offset
where date_month between addDate(start_date, INTERVAL -offset*12 MONTH) 
    and addDate(end_date, INTERVAL -offset*12 MONTH)
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
group by 2
```
<!-- where date_month between addDate(start_date, INTERVAL -offset*toInt8({{interval_granularity.selected}}) MONTH) 
                and addDate(end_date, INTERVAL -offset*toInt8({{interval_granularity.selected}}) MONTH) -->
```sql restrictions_total_daus
select
SUM(CASE WHEN offset = 0 THEN daily_active_users END) AS current_daily_active_users,
case when SUM(CASE WHEN offset = 1 THEN daily_active_users END) > {{$privacy_restriction_value_dau}} 
then SUM(CASE WHEN offset = 1 THEN daily_active_users END)
else null end AS prior_daily_active_users
from {{metric_calculation_total_daus}} as metric_calculation
```


{% if
    data="restrictions_total_daus"
    where="current_daily_active_users > {{$privacy_restriction_value_dau}}"
    %}

    {% big_value
        data="restrictions_total_daus"
        value="current_daily_active_users"
        title="Total Daily Active Users"
        info="The number of daily interactions with the care team, including consultations and chats."
        comparison={
            compare_vs="target"
            target="prior_daily_active_users"
            delta=false
            display_type="compared_value"
            text="in previous year"
        }
    /%}

{% /if %}

{% else %}

    {% callout
    type="warning"
    %}
    Total Daily Active Users cannot be displayed because it is below privacy threshold
    {% /callout %}

{% /else %}
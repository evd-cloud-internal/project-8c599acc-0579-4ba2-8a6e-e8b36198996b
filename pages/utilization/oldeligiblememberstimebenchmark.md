---
name: OLD_eligible_members_time_benchmark
assetId: 324c5af3-ccb0-4736-b7ce-6aba71ffee54
type: partial
---

---
privacy_restriction_value_elig: 35
---
<!-- {% partial
    file="combinefilters"
/%} -->
```sql metric_desc_elig_member
select
    value as description
from base_client_reporting_metric_descriptions
where metric_name = 'Number of Eligible Members'
    and language = 'EN'
```

```sql ranked_monthly
WITH
    toDate({{date_start.selected}}) AS start_date,
    toDate({{date_end.selected}}) AS end_date
    select
        toMonth(date_month) as month
        , offset
        , sum(max_eligible_members) as eligible_members
    from client_reporting_monthly
    ARRAY JOIN range(0, 2) AS offset
    where date_month between addDate(start_date, INTERVAL -offset*12 MONTH) 
    and addDate(end_date, INTERVAL -offset*12 MONTH)
    and {{filter_select.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    group by 1,2
    order by 1 desc
    limit 2
```
    <!-- where date_month between addDate(start_date, INTERVAL -offset*toInt8({{interval_granularity.selected}}) MONTH) 
                and addDate(end_date, INTERVAL -offset*toInt8({{interval_granularity.selected}}) MONTH) -->

```sql metric_calculation_elig_member
select
    offset,
    sum(eligible_members) as eligible_members
from {{ranked_monthly}}
group by 1
```

```sql restrictions_elig_member
select
SUM(CASE WHEN offset = 0 THEN eligible_members END) AS current_eligible_members,
case when 
SUM(CASE WHEN offset = 1 THEN eligible_members END) > {{$privacy_restriction_value_elig}} 
then SUM(CASE WHEN offset = 1 THEN eligible_members END)
else null end AS prior_eligible_members
from {{metric_calculation_elig_member}} as metric_calculation
```


{% if
    data="restrictions_elig_member"
    where="current_eligible_members > {{$privacy_restriction_value_elig}}"
%}

    {% big_value
        data="restrictions_elig_member"
        value="current_eligible_members"
        title="Total Eligible Members"
        info="The number of employees eligible for Dialogue, as of the last day of the latest month."
        comparison={
            compare_vs="target"
            target="prior_eligible_members"
            display_type="compared_value"
            text="in previous year"
            delta=false
        }
    /%}

{% /if %}

{% else %}

{% callout
type="warning"
%}
Metric value(s) cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else %}





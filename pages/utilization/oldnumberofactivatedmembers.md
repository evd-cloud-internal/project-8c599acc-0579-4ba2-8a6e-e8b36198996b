---
name: OLD_number_of_activated_members
assetId: 73882e3d-6aa3-4f98-8024-c004cbf1b9c5
type: partial
---

---
privacy_restriction_value: 5
---
```sql metric_desc_activated_member
select
    value as description
from base_client_reporting_metric_descriptions
where language = 'EN'
    and metric_name = 'Number of Activated Members'
```

```sql metric_calculation_activated_member
with month_ranked as (
select
    date_month
    , 'Member(s)' as type
    , sum(cast(max_activated_employees as decimal(10,2))) as metric
from client_reporting_monthly
where date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and date_month < date_trunc('month',today())

    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
order by date_month desc
limit 1

union all

select
    date_month
    , 'Family Member(s)' as type
    , sum(cast(max_activated_family as decimal(10,2))) as metric
from client_reporting_monthly
where date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and date_month < date_trunc('month',today())

    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
order by date_month desc
limit 1
)
select
type
, metric
from month_ranked
```

```sql restrictions_activated_member
select
type
, metric
from {{metric_calculation_activated_member}} as metric_calculation
where metric >= {{$privacy_restriction_value}}
```


{% if
    data="restrictions_activated_member"
    condition="has_rows"
%}

<!-- {% row
card=true 
%} -->
    {% combo_chart
        data="restrictions_activated_member"
        x="type"
        y_fmt="num2k"
        title="Number of Activated Members"
        info="The number of members that have used Dialogue at least once."
        x_axis_options={
            gridlines=false
            title=""
        }
        y_axis_options={
            gridlines=false
        }
    %}
    {% bar
        y="metric as _"
        axis="y1"
        data_labels={
            position="above"
        }
    /%}
    {% /combo_chart %}

{% /if %}

{% else %}

{% callout
type="warning"
%}
Number of Activated Members cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else %}




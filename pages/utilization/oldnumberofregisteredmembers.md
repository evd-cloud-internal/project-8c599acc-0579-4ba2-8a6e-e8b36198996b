---
name: OLD_number_of_registered_members
assetId: 5cd95f90-385b-4b9d-9541-2581ef34f4d8
type: partial
---

---
privacy_restriction_value: 5
---
<!-- {% partial
    file="combinefilters"
/%} -->
```sql metric_desc_registered_member
select
    value as description
from base_client_reporting_metric_descriptions
where language = 'EN'
    and metric_name = 'Number of Registered Members'
```

```sql metric_calculation_registered_member
with month_ranked as (
select
    date_month
    , 'Member(s)' as type
    , sum(cast(max_registered_employees as decimal(10,2))) as metric
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
    , sum(cast(max_registered_family as decimal(10,2))) as metric
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
<!-- {% table
    data="metric_calculation_registered_member"
%}
{% /table %} -->
```sql restrictions_registered_member
select
distinct type
, metric
from {{metric_calculation_registered_member}} as metric_calculation
where metric >= {{$privacy_restriction_value}}
```

{% if
    data="restrictions_registered_member"
    condition="has_rows"
%}

<!-- {% row card=true %} -->
    {% combo_chart
        data="restrictions_registered_member"
        x="type"
        y_fmt="num2k"
        title="Number of Registered Members"
        info="The number of members that have completed sign-up."
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
Number of Registered Members cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else %}






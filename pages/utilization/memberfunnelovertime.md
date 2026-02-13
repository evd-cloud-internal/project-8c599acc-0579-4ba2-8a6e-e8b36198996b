---
name: member_funnel_overtime
assetId: 2fda6730-8b64-460e-9943-b8887596b2e3
type: partial
---

```sql metric_calculation_member_funnel_overtime
with monthly as (
select
    date_month
    , sum(max_eligible_members) as eligible_members
    , sum(max_registered_employees) as max_registered_employees
    , sum(max_activated_employees) as max_activated_employees
    , row_number() over (order by date_month desc) as rank_month
from client_reporting_monthly
where date_month >= {{date_start.selected}}
    and date_month <= {{date_end.selected}}
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
)

select date_month
    , eligible_members
    , round(toFloat64(max_registered_employees)/ toFloat64(eligible_members), 3) as registration_rate
    , round(toFloat64(max_activated_employees)/ toFloat64(max_registered_employees), 3) as activation_rate
    , max_registered_employees
    , max_activated_employees
from monthly
where rank_month between 1 and 12
```

```sql restrictions_member_funnel_overtime
select
date_month
    , case
        when eligible_members > {{$privacy_restriction_high}} then eligible_members
        else null
    end as eligible_members
    , case
        when eligible_members > {{$privacy_restriction_high}} and max_registered_employees >= {{$privacy_restriction_low}} then registration_rate
        else null
    end as registration_rate
    , case
        when eligible_members > {{$privacy_restriction_high}} and max_registered_employees >= {{$privacy_restriction_low}} and max_activated_employees >= {{$privacy_restriction_low}} then activation_rate
        else null
    end as activation_rate
from {{metric_calculation_member_funnel_overtime}} as metric_calculation
```

```sql full_restrictions_check_member_funnel_overtime
select
eligible_members
, registration_rate
, activation_rate
from {{restrictions_member_funnel_overtime}}
where eligible_members is not null
    or registration_rate is not null
    or activation_rate is not null
```

{% if
    data="full_restrictions_check_member_funnel_overtime"
    condition="has_rows"
    %}


{% row card=true %}

    {% combo_chart
        data="restrictions_member_funnel_overtime"
        x="date_month"
        y2_fmt="pct1"
        title="{{$translations.metric_definitions.utilization.activation_funnel_over_time.title}}"
        info="{{$translations.metric_definitions.utilization.activation_funnel_over_time.description}}"
        info_link="#{{$translations.metric_definitions.utilization.activation_funnel_over_time.info_link}}"
        info_link_title="{{ $translations.read_more}}"
        x_fmt="mmmm"
        x_axis_options={
            min_interval="month"
            gridlines=false
            title=""
        }
        y_axis_options={
            gridlines=false
            title=""

        }
        y2_axis_options={
            gridlines=false
            title=""
        }
        chart_options={
            top_padding=5
        }

    %}
        {%bar y="eligible_members as `{{$translations.metric_definitions.utilization.activation_funnel_over_time.eligible_members}}`"
            axis="y1"
            fmt="num1k"
            data_labels={
                position="above"
            }
        /%}
        {%line y="registration_rate as `{{$translations.metric_definitions.utilization.activation_funnel_over_time.registration_rate}}`"  
            axis="y2"
            options={
                markers={shape="emptyCircle"}
                width=3
            }
            data_labels={
                position="above"
                border_color="#00000000"

            }
        /%}
        {%line y="activation_rate as `{{$translations.metric_definitions.utilization.activation_funnel_over_time.activation_rate}}`"  
            axis="y2"
            options={
                markers={shape="emptyCircle"}
                width=3
            }
            data_labels={
                position="below"
                border_color="#00000000"
            }
        /%}
    {% /combo_chart %}

{% /row %}

{% /if %}

{% else_if
    data="full_restrictions_check_member_funnel_overtime"
    condition="no_rows"
    %}

    {% callout
        type="warning"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.activation_funnel_over_time.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /else_if %}



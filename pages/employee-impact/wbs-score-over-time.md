---
name: WBS score over time
assetId: b17b87bf-2ce5-4060-b0ab-683bce04d0b0
type: partial
---

```sql metric_calculation_wbs_over_time
with wbs as (
select
    wbs_questionnaires_with_contracts.qnaire_id as qnaire_tid
    , wbs_questionnaires_with_contracts.user_id
    , score
    , wbs_questionnaires_with_contracts.mood_score
    , wbs_questionnaires_with_contracts.stress_score
    , wbs_questionnaires_with_contracts.activeness_score
    , wbs_questionnaires_with_contracts.sleep_score
    , wbs_questionnaires_with_contracts.purpose_score
    , wbs_total_score
    , wbs_completed_at_est
    , date_trunc('day', wbs_completed_at_est) as date_day_est
    , date_trunc('month', wbs_completed_at_est) as date_month_est
    , wbs_questionnaires_with_contracts.reporting_group_id
from wbs_questionnaires_with_contracts
where date_month_est >= {{date_start.selected}}
and date_month_est <= {{date_end.selected}}
and date_month_est < date_trunc('month',today())
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
)

, unique_active_users as (
    select date_month
        , count(distinct user_id) as total_active_users
    from active_users
    where date_month >= {{date_start.selected}}
    and date_month <= {{date_end.selected}}
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and date_month < date_trunc('month',today())
    group by 1
)

, monthly_wbs_score as (
select date_month_est
    , count(distinct user_id) as total_wbs_users
    , avg(wbs_total_score) as avg_wbs_score
    , count(distinct qnaire_tid) as total_qnaire_tid
    from wbs
    group by 1
)

, ranked as (
select unique_active_users.date_month as date_month
    , total_wbs_users / total_active_users as participation_rate
    , total_wbs_users as total_users
    , avg_wbs_score as average_score
    , total_qnaire_tid
    , rank() over (order by unique_active_users.date_month desc) as rank
from unique_active_users
inner join monthly_wbs_score
    on unique_active_users.date_month = monthly_wbs_score.date_month_est
)

select
*
from ranked
where rank <= 10
```


```sql restrictions_wbs_over_time
select
date_month
, participation_rate
, average_score
, total_qnaire_tid
from {{metric_calculation_wbs_over_time}} as metric_calculation
where total_qnaire_tid >= {{$privacy_restriction_low}}

```

{% stack card=true %}

    ### {{$translations.section_descriptions.employee_impact.wellbeing_score.title}}
    <!-- Wellbeing Score -->
    <!-- Indicates overall member well-being at a given point in time, tracking shifts in self-reported health perception. -->
    {{$translations.section_descriptions.employee_impact.wellbeing_score.overview}}

    {% if
        data="restrictions_wbs_over_time"
        condition="has_rows"
        %}

        {% combo_chart
            data="restrictions_wbs_over_time"
            x="date_month"
            title="{{$translations.metric_definitions.employee_impact.avg_wbs_over_time.title}}"
            x_fmt="mmm"
            y_fmt="num0"
            y2_fmt="pct1"
            x_axis_options={
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
            info="{{$translations.metric_definitions.employee_impact.avg_wbs_over_time.description}}"
            info_link="#{{$translations.metric_definitions.employee_impact.avg_wbs_over_time.info_link}}"
            info_link_title="{{$translations.read_more}}"
            chart_options={
                top_padding=5
            }
        %}
            {%line y="average_score as `{{$translations.metric_definitions.employee_impact.avg_wbs_over_time.average_score}}`" 
                axis="y1" 
                data_labels={
                    position="above"
                }
                options={
                    markers={shape="circle"}
                    width=3
                }
            /%}
            {%line y="participation_rate as `{{$translations.metric_definitions.employee_impact.avg_wbs_over_time.participation_rate}}`"  
                axis="y2"
                data_labels={position="below"}
                options={
                    markers={shape="circle"}
                    width=3
                }
            /%}
        {% /combo_chart %}

    {% /if %}

    {% else %}

            {% callout
                type="warning"
                %}

                {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.avg_wbs_over_time.title}}{{$translations.privacy_disclaimer_end}}
            
            {% /callout %}

    {% /else %}

{% /stack %}


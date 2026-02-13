---
name: monthly_engagement_rate
assetId: 2613d139-722e-47f6-ab3a-474d64f91b68
type: partial
---

```sql metric_calculation_monthly_engage
with rank as (
select
    date_month
    , sum(daus) as daily_active_users
    , sum(avg_eligible_members) as total_avg_eligible_members
    , sum(daus) / toFloat64(sum(avg_eligible_members)) as engagement_rate
    , rank() over (order by date_month desc) as rank
from client_reporting_monthly
where date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and date_month < date_trunc('month',today())
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
)
select
*
from rank
where rank <= 12
```

```sql restrictions_monthly_engage
select
date_month
, daily_active_users
from {{metric_calculation_monthly_engage}} as metric_calculation
where daily_active_users >= {{$privacy_restriction_low}}
```

{% stack card=true %}

    ### {{$translations.section_descriptions.utilization.engagement_2.title}}
    <!-- Engagement -->
    <!-- Indicates how frequently employees return, explore, or engage with core features, revealing overall adoption health. -->
    {{$translations.section_descriptions.utilization.engagement_2.overview}}

    {% if
        data="restrictions_monthly_engage"
        condition="has_rows"
    %}

            {% combo_chart
                data="metric_calculation_monthly_engage"
                x="date_month"
                title="{{$translations.metric_definitions.utilization.monthly_engagement_rate.title}}"
                x_fmt="mmm"
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
                info="{{$translations.metric_definitions.utilization.monthly_engagement_rate.description}}"
                info_link="#{{$translations.metric_definitions.utilization.monthly_engagement_rate.info_link}}"
                info_link_title="{{ $translations.read_more}}"
                chart_options={
                    top_padding=5
                }
            %}
                {%bar y="daily_active_users as `{{$translations.metric_definitions.utilization.monthly_engagement_rate.daily_active_users}}`" 
                    axis="y1" 
                    data_labels={position="above"}
                /%}

                {%line y="engagement_rate as `{{$translations.metric_definitions.utilization.monthly_engagement_rate.engagement_rate}}`"
                    axis="y2"
                    data_labels={position="above" border_color="#00000000"}
                    options={
                        markers={shape="emptyCircle"}
                        width=3
                    }
                /%}

            {% /combo_chart %}

    {% /if %}

    {% else_if
        data="restrictions_monthly_engage"
        condition="no_rows"
        %}

            {% callout
            type="warning"
            %}

            {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.monthly_engagement_rate.title}}{{$translations.privacy_disclaimer_end}}

            {% /callout %}

    {% /else_if %}

{% /stack %}


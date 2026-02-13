---
name: satisfaction_over_time
assetId: 52650436-8e4c-44f7-9bad-d6b8324b4721
type: partial
---

```sql metric_calculation_satisfaction_over_time
with nps_patient_survey_program_mapping as (
select
    case
        when {{program_filter.selected}} = 'all' then 'all'
        else program
    end as program
    , date_month, score, surveys
from nps_patient_survey
where program is not null
and program not in ('wellness', 'mso')
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
and date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and date_month < date_trunc('month', today())
)

, agg as (
select
date_month
, avg(score) as avg_score
, sum(surveys) as surveys_count
from nps_patient_survey_program_mapping
where program = {{program_filter.selected}}

group by 1
)
, ranked as (
select
*
, rank() over (order by date_month desc) as rank
from agg
)
select
*
from ranked
where rank <=12
```

```sql restrictions_satisfaction_over_time
select
    date_month
    , avg_score
    , surveys_count
from {{metric_calculation_satisfaction_over_time}} as metric_calculation
where surveys_count >= {{$privacy_restriction_low}}
```

{% stack card=true %}

    ### {{$translations.section_descriptions.satisfaction.satisfaction_score.title}}
    <!-- Satisfaction Score -->
    <!-- Indicates overall satisfaction with the care experience and percieved service quality. -->
    {{$translations.section_descriptions.satisfaction.satisfaction_score.overview}}

    {% if
        data="restrictions_satisfaction_over_time"
        condition="has_rows"
        where="surveys_count >= {{$privacy_restriction_low}}"
        %}

        {% combo_chart
            data="restrictions_satisfaction_over_time"
            x="date_month"
            title="{{$translations.metric_definitions.satisfaction.satisfaction_over_time.title}}"
            info="{{$translations.metric_definitions.satisfaction.satisfaction_over_time.description}}"
            x_fmt="mmm"
            x_axis_options={gridlines=false title=""}
            y_axis_options={gridlines=false}
            %}

            {% line
                y="avg_score as `{{$translations.metric_definitions.satisfaction.satisfaction_over_time.satisfaction_score}}`"
                axis="y1"
                data_labels={position="below"}
                options={
                    markers={
                        shape="circle"
                        size=6
                    }
                }
            /%}

            {% /combo_chart %}

    {% /if %}

    {% else %}

        {% callout type="warning" %}

            {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.satisfaction.satisfaction_over_time.title}}{{$translations.privacy_disclaimer_end}}

        {% /callout %}

    {% /else %}

{% /stack %}



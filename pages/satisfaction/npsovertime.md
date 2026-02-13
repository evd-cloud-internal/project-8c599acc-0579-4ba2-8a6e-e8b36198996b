---
name: nps_over_time
assetId: dde314f1-a59b-464e-a84d-aab87fe12836
type: partial
---

```sql metric_calculation_nps_over_time
with nps_patient_survey_program_mapping as (
select
    case
        when {{program_filter.selected}} = 'all' then 'all'
        else program
    end as program
    , *
from nps_patient_survey
where program is not null
and program not in ('wellness', 'mso')   
)

, agg as (
select
date_month
, 100*(sum(promoters) - sum(detractors)) / sum(surveys) as nps
, sum(surveys) as surveys_count
from nps_patient_survey_program_mapping
where date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and program = {{program_filter.selected}}
and date_month < date_trunc('month', today())
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
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

```sql restrictions_nps_over_time
select
    date_month
    , nps
    , surveys_count
from {{metric_calculation_nps_over_time}} as metric_calculation
where surveys_count >= {{$privacy_restriction_low}}
```

{% stack card=true %}

    ### {{$translations.section_descriptions.satisfaction.net_promoter_score.title}}
    <!-- Net Promoter Score -->
    <!-- Indicates how likely members are to recommend Dialogue's services (a measure of trust and loyalty). -->
    {{$translations.section_descriptions.satisfaction.net_promoter_score.overview}}

    {% if
        data="restrictions_nps_over_time"
        condition="has_rows"
        where="surveys_count >= {{$privacy_restriction_low}}"
        %}

            {% combo_chart
                data="restrictions_nps_over_time"
                x="date_month"
                title="{{$translations.metric_definitions.satisfaction.nps_over_time.title}}"
                info="{{$translations.metric_definitions.satisfaction.nps_over_time.description}}"
                x_fmt="mmm"
                x_axis_options={gridlines=false title=""}
                y_axis_options={gridlines=false}
                %}

                {% line
                    y="nps as `{{$translations.metric_definitions.satisfaction.nps_over_time.nps}}`"
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

            {% callout
            type="warning"
            %}
            {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.satisfaction.nps_over_time.title}}{{$translations.privacy_disclaimer_end}}

            {% /callout %}

    {% /else %}

{% /stack %}




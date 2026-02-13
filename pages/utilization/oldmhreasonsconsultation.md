---
name: OLD_mh_reasons_consultation
assetId: 5cc9cbf7-eaf5-4d94-a082-cac1cfb5cd0c
type: partial
---

---
privacy_restriction_value: 10
<!-- reporting_block_id: dia -->
---

<!-- {% partial
    file="combinefilters"
/%} -->
```sql metric_desc_mh_reason_for_consult
select
value as description 
from base_client_reporting_metric_descriptions 
where language = 'EN'
and metric_name = 'Reasons for Mental Health Consultations'
```

```sql metric_calculation_mh_reason_for_consult
with reasons_for_consult as (
select
  mh_reason_for_consult_en as mh_reason_for_consult
    , count(distinct episode_id) as cases
    , sum(cases) over () as total_cases
from episodes_with_contracts
where date_month_est >= {{date_start.selected}}
    and date_month_est <= {{date_end.selected}}
    [[and program = {{program_filter.selected}}]]
    and is_suitable_episode
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and mh_reason_for_consult_en is not null
group by mh_reason_for_consult_en
)
select
    mh_reason_for_consult
    , cases / total_cases as proportion
    , cases
    , total_cases
from reasons_for_consult
```

```sql restrictions_mh_reason_for_consult
select
    mh_reason_for_consult
    , proportion
    , total_cases
from {{metric_calculation_mh_reason_for_consult}} as metric_calculation
where cases >= {{$privacy_restriction_value}}
```

{% if
    data="restrictions_mh_reason_for_consult"
    condition="has_rows"
%}

{% row card=true %}

{% bar_chart
    data="restrictions_mh_reason_for_consult"
    x="proportion"
    y="mh_reason_for_consult"
    x_fmt="pct"
    title="Reasons for Mental Health Consultations"
    order="proportion asc"
    info="The breakdown of reasons for mental health consultations, as indicated by the mental health specialist at initial assessment. Please note that due to the nature of sleep issues, the percentages reported may be lower for clients enrolled in our Primary Care program."
    y_axis_options={
        gridlines=false
        title=""
    }
    x_axis_options={
        label_wrap=true
        gridlines=false
        title=""
        }
/%}

{% /row %}


{% /if %}

{% else %}

{% callout
type="warning"
%}
Reasons for Mental Health Consultations cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else %}
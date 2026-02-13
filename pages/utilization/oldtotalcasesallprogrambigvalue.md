---
name: OLD_total_cases_all_program_big_value
assetId: 70700c70-04c2-4475-88a1-fcc1a8b1117c
type: partial
---

---
privacy_restriction_value_cases_total: 10
privacy_restriction_value_sessions: 5
reporting_block_id: dia
---
<!-- {% partial
    file="combinefilters"
/%} -->
```sql metric_desc_total_cases_exec_summary
select
    value as description
from base_client_reporting_metric_descriptions
where metric_name = 'Total Cases (exec summary)'
    and language = 'EN'
```

    <!-- toInt8({{number_of_previous_periods.selected}}) AS n, -->
```sql client_reporting_time_gran
WITH
    toDate({{date_start.selected}}) AS start_date,
    toDate({{date_end.selected}}) AS end_date
select 
    date_month,
    sum(cases) as cases,
    sum(sessions) as sessions,
    
    sum(cases_eap) as cases_eap,
    sum(cases_mental_health) as cases_mental_health,
    sum(cases_primary_care) as cases_primary_care,
    
    sum(sessions_eap) as sessions_eap,
    sum(sessions_mental_health) as sessions_mental_health,
    sum(sessions_primary_care_utilization_rate) as sessions_primary_care
from client_reporting_monthly as d
where ((date_month between addMonths(start_date, -12) and addMonths(end_date, -12))
    or (date_month between start_date and end_date))
    and {{filter_select.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
order by 1
```
<!-- {% table
    data="client_reporting_time_gran"
%}
{% /table %} -->
<!-- where date_month between addDate(start_date, INTERVAL -offset*12 MONTH) 
                and addDate(end_date, INTERVAL -offset*12 MONTH) -->
<!-- where date_month between addDate(start_date, INTERVAL -offset*toInt8({{interval_granularity.selected}}) MONTH) 
                and addDate(end_date, INTERVAL -offset*toInt8({{interval_granularity.selected}}) MONTH) -->
<!-- where date_month between addDate(start_date, INTERVAL -offset {{interval_granularity.literal}}) 
                and addDate(end_date, INTERVAL -offset {{interval_granularity.literal}}) -->
```sql big_values_overview
WITH
    toDate({{date_start.selected}}) AS start_date,
    toDate({{date_end.selected}}) AS end_date
SELECT
  SUM(CASE WHEN date_month between start_date and end_date THEN cases END) AS current_cases,
  SUM(CASE WHEN date_month between start_date and end_date THEN sessions END) AS current_sessions,

  CASE 
    WHEN SUM(CASE WHEN date_month between addMonths(start_date, -12) and addMonths(end_date, -12) THEN cases END) >= 
         {{$privacy_restriction_value_cases_total}}
    THEN SUM(CASE WHEN date_month between addMonths(start_date, -12) and addMonths(end_date, -12) THEN cases END)
    ELSE NULL
  END AS prior_cases,

  CASE 
    WHEN SUM(CASE WHEN date_month between addMonths(start_date, -12) and addMonths(end_date, -12) THEN sessions END) >= 
         {{$privacy_restriction_value_sessions}}
    THEN SUM(CASE WHEN date_month between addMonths(start_date, -12) and addMonths(end_date, -12) THEN sessions END)
    ELSE NULL
  END AS prior_sessions

FROM {{client_reporting_time_gran}};
```

```sql restrictions_total_cases_overall
select
current_cases
from {{big_values_overview}} as big_values
```

{% if
    data="restrictions_total_cases_overall"
    where="current_cases >= {{$privacy_restriction_value_cases_total}}"
%}

<!-- {% row card=true %} -->
{% big_value
    data="big_values_overview"
    value="current_cases"
    comparison={
        compare_vs="target"
        target="prior_cases"
        text="vs. last year"
        display_type="pct"
        delta=true
        down_is_good=false
        pct_fmt="pct1"
    }
    info="The total number of cases initiated across all programs."
    title="Total Cases"
/%}

<!-- {% modal
    title="info"
    icon="info"
    icon_only=true
    variant="ghost"
%}
{% partial
    file="benchmarking/lastperiodinfomodal"
/%}
{% table
    data="metric_desc_total_cases_exec_summary"
%}
{% /table %}
{% /modal %}
{% /row %} -->

{% /if %}

{% else %}

{% callout
type="warning"
%}
Metric value(s) cannot be displayed because it is below privacy threshold
{% /callout %}

{% /else %}
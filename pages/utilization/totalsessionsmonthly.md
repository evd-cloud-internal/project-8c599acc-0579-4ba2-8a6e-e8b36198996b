---
name: total_sessions_monthly
assetId: 7ebc0ec7-1a50-4635-b560-d497bfdb2d2a
type: partial
---

```sql metric_calculation_monthly_total_sessions
with client_report_time as (
select 
    date_month
    , sum(sessions) as sessions_all
    , sum(sessions_eap) as sessions_eap

    , sum(sessions_mental_health) as sessions_mental_health

    , sum(sessions_primary_care) as sessions_primary_care

from client_reporting_monthly
where {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and (date_month between {{date_start.selected}} and {{date_end.selected}}
    or (date_month between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12)))
group by 1
)

select
  date_month
  , toYear(date_month) as year
  , case 
  when sessions_{{program_filter.literal}} >= {{$privacy_restriction_low}}
  then cast(sessions_{{program_filter.literal}} as int)
  else null
  end as sessions
from client_report_time
where sessions >= {{$privacy_restriction_low}}
```

```sql restriction_calculation_monthly_total_sessions
select * 
from {{metric_calculation_monthly_total_sessions}} 
where date_month between {{date_start.selected}} and {{date_end.selected}}
```



{% stack card=true %}

  <!-- ### sessions
  Volume of member journeys where the member has gotten value from a Dialogue service. -->

  {% if
    data="restriction_calculation_monthly_total_sessions"
    condition="has_rows"
    where="date_month between {{date_start.selected}} and {{date_end.selected}}"
    %}
    {% bar_chart
      data="metric_calculation_monthly_total_sessions"
      x="date_month"
      y="sessions as _"
      data_labels={
        position="middle"
        border_color="#00000000"
      }
      x_fmt="mmm"
      y_fmt="num0"
      info="{{$translations.metric_definitions.utilization.monthly_sessions.description}}"
      info_link="#{{$translations.metric_definitions.utilization.monthly_sessions.info_link}}"
      info_link_title="{{ $translations.read_more}}"
      x_axis_options={
        gridlines=false
        title=""
      }
      y_axis_options={
        gridlines=false
      }
      title="{{$translations.metric_definitions.utilization.monthly_sessions.title}}"
      series="year"
      stacked=false
      date_grain="month of year"
      order="year asc"
    /%}

  {% /if %}

  {% else %}

    {% callout
      type="warning"
      %}

      {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.monthly_sessions.title}} {{program_filter.label}}{{$translations.privacy_disclaimer_end}}

    {% /callout %}

  {% /else %}

{% /stack %}


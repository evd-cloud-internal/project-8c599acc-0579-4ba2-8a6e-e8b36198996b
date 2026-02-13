---
name: total_cases_monthly
assetId: 576d3181-0fb2-41a0-a404-9d18f6a49b09
type: partial
---

```sql metric_calculation_monthly_total_cases
with client_report_time as (
select 
    date_month
    , sum(cases) as cases_all
    , sum(cases_eap) as cases_eap

    , sum(cases_mental_health) as cases_mental_health

    , sum(cases_primary_care) as cases_primary_care

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
  when cases_{{program_filter.literal}} >= {{$privacy_restriction_low}}
  then cast(cases_{{program_filter.literal}} as int)
  else null
  end as cases

from client_report_time
where cases >= {{$privacy_restriction_low}}
```


{% stack card=true %}

  <!-- ### Cases
  Volume of member journeys where the member has gotten value from a Dialogue service. -->

  {% if
    data="metric_calculation_monthly_total_cases"
    condition="has_rows"
    where="date_month between {{date_start.selected}} and {{date_end.selected}}"
    %}
    {% bar_chart
      data="metric_calculation_monthly_total_cases"
      x="date_month"
      y="cases as _"
      data_labels={
        position="middle"
        border_color="#00000000"
      }
      x_fmt="mmm"
      y_fmt="num0"
      info="{{$translations.metric_definitions.utilization.monthly_cases.description}}"
      info_link="#{{$translations.metric_definitions.utilization.monthly_cases.info_link}}"
      info_link_title="{{ $translations.read_more}}"
      x_axis_options={
        gridlines=false
        title=""
      }
      y_axis_options={
        gridlines=false
      }
      title="{{$translations.metric_definitions.utilization.monthly_cases.title}}"
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

      {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.monthly_cases.title}} {{program_filter.label}}{{$translations.privacy_disclaimer_end}}

    {% /callout %}

  {% /else %}

{% /stack %}


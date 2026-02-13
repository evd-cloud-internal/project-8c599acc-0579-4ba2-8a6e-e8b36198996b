---
name: top_reason_for_consult
assetId: fbf07ece-3d2e-46f9-bb29-a5f39efcfeb2
type: partial
---

```sql metric_calculation_reason_for_consult
with episodes_with_contracts_program_mapping as (
select
    case
        when {{program_filter.selected}} = 'all' then 'all'
        else program
    end as program
    , *
from episodes_with_contracts
where program is not null
and program not in ('wellness', 'mso')
and issue_type_reporting_en not in ('RX renewal')
)
, reasons_for_consult as (
select
  issue_type_reporting_{{$translations.inline_query}} as reason_for_consult
  , program

  , sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between {{date_start.selected}} and {{date_end.selected}} then 1 end) as cases_reasons_for_consult_current

  , case when 
  sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then 1 end) >= {{$privacy_restriction_medium}} 
  then sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then 1 end) 
  else null 
  end as cases_reasons_for_consult_previous

  , sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between {{date_start.selected}} and {{date_end.selected}} then 1 end) / sum(sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between {{date_start.selected}} and {{date_end.selected}} then 1 end)) over () as percent_current

  , case when 
  sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then 1 end) >= {{$privacy_restriction_medium}} 
  then sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then 1 end) / sum(sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then 1 end)) over () 
  else null
  end as percent_previous

  , sum(sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between {{date_start.selected}} and {{date_end.selected}} then 1 end)) over () as total_cases_current
  , case when 
  sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then 1 end) >= {{$privacy_restriction_medium}} 
  then sum(sum(case when issue_type_reporting_{{$translations.inline_query}} is not null and date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then 1 end)) over () 
  else null
  end as total_cases_previous

from episodes_with_contracts_program_mapping
where (date_month_est between {{date_start.selected}}
    and {{date_end.selected}}
    or date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12))
    and program = {{program_filter.selected}}
    and is_suitable_episode
    and issue_type_reporting_{{$translations.inline_query}} <> 'n/a'
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
group by issue_type_reporting_{{$translations.inline_query}}, program
order by cases_reasons_for_consult_current desc
)
select
reason_for_consult
, program
, percent_current
, percent_previous
, total_cases_current
, total_cases_previous
, cases_reasons_for_consult_current
, cases_reasons_for_consult_previous
from reasons_for_consult
where percent_current > 0
order by percent_current desc
limit 10
```

```sql restrictions_reason_for_consult
select
reason_for_consult
, program
, coalesce(round(percent_current / sum(percent_current) over (),2),0) as percent_masked_current
, total_cases_current
, cases_reasons_for_consult_current
, coalesce(round(percent_previous / sum(percent_previous) over (),2),0) as percent_masked_previous
, total_cases_previous
, cases_reasons_for_consult_previous
from {{metric_calculation_reason_for_consult}} as metric_calculation
where cases_reasons_for_consult_current >= {{$privacy_restriction_medium}}
```

{% if
    data="restrictions_reason_for_consult"
    condition="has_rows"
    %}
    <!-- {% partial
        file="utilization/consultationpercase"
        variables={
            reporting_block_id=$reporting_block_id
        }
    /%} -->
    <!-- {% row card=true%} -->
    {% table
        data="restrictions_reason_for_consult"
        order="percent_masked_current desc"
        title="{{$translations.metric_definitions.utilization.reason_for_consult.title}}"
        info="{{$translations.metric_definitions.utilization.reason_for_consult.description}}"
        info_link="#{{$translations.metric_definitions.utilization.reason_for_consult.info_link}}"
        info_link_title="{{ $translations.read_more}}"
    %}
    {% dimension
        value="reason_for_consult"
        title="{{$translations.metric_definitions.utilization.reason_for_consult.reason}}"
    /%}

    {% dimension
        value="cases_reasons_for_consult_current"
        title="{{$translations.metric_definitions.utilization.reason_for_consult.cases}}"
        fmt="num"
    /%}

    {% measure
        value="cases_reasons_for_consult_current"
        comparison={
            compare_vs="target"
            target="cases_reasons_for_consult_previous"
            pct_fmt="pct1"
        }
        title=""
        viz="delta"
    /%}

    {% dimension
        value="percent_masked_current"
        title="{{$translations.metric_definitions.utilization.reason_for_consult.percent_cases}}"
        fmt="pct1"
        info="{{$translations.metric_definitions.utilization.reason_for_consult.percent_cases_description}}"
    /%}

    {% measure
        value="percent_masked_current"
        fmt="pct1"
        comparison={
            compare_vs="target"
            target="percent_masked_previous"
            pct_fmt="pct1"
        }
        title=""
        viz="delta"
    /%}
    {% /table %}
    <!-- {% bar_chart
        data="restrictions_reason_for_consult"
        x="reason_for_consult"
        y="percent_masked as _"
        y_fmt="pct1"
        title="Top Reasons for Consult"
        data_labels={position="above"}
        bar_options={color="#C4E6F1"}
        order="percent_masked desc"
        having="percent_masked > 0"
        limit=10
        info="The top three reasons for consulting."
        y_axis_options={
            gridlines=false
            title=""
        }
        x_axis_options={
            label_wrap=true
            gridlines=false
            title=""
            }
    /%} -->
        <!-- {% modal
            title="Info"
            icon_only=true
            variant="ghost"
            icon="info"
        %}
        {% table
            data="metric_desc_reason_for_consult"
            dimensions=["description"]
        /%}
        {% /modal %}
    {% /row %} -->

{% /if %}

{% else %}

{% callout
type="warning"
%}
{{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.reason_for_consult.title}}{{$translations.privacy_disclaimer_end}}
{% /callout %}

{% /else %}

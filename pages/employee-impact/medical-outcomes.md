---
name: medical outcomes
assetId: 65f485b0-8c14-40b0-ac6c-610ed044f416
type: partial
---

```sql metric_calculation_medical_outcomes
with case_outcomes as (
select
    outcome_reporting_{{$translations.inline_query}} as outcome
    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then episode_id end) as cases_outcome
    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then episode_id end) as cases_outcome_last_year
    , sum (cases_outcome) over () as total_cases
    , sum (cases_outcome_last_year) over () as total_cases_last_year
from episodes_with_contracts
where date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and {{date_end.selected}}
and date_month_est < date_trunc('month',today())
and is_suitable_episode
and outcome_reporting_{{$translations.inline_query}} is not null
and outcome_reporting_{{$translations.inline_query}} <> ''
and issue_type_reporting_{{$translations.inline_query}} is not null
and program = {{program_filter.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
order by 2 desc
)

select *
    , cases_outcome / total_cases as percent_cases
    , cases_outcome_last_year / total_cases_last_year as percent_cases_last_year
from case_outcomes
```

```sql restriction_calculation_medical_outcomes
with apply_restrictions as (
select outcome
    , case when cases_outcome < {{$privacy_restriction_low}} then null
    else percent_cases end as percent_cases
    , case when cases_outcome < {{$privacy_restriction_low}} then null
    else cases_outcome end as cases_outcome
    , case when cases_outcome_last_year < {{$privacy_restriction_low}} then null
    else percent_cases_last_year end as percent_cases_last_year
    , case when cases_outcome_last_year < {{$privacy_restriction_low}} then null
    else cases_outcome_last_year end as cases_outcome_last_year
    , sum(cases_outcome) over () as total_cases
from {{metric_calculation_medical_outcomes}}
)

select *
from apply_restrictions
where cases_outcome > 0
```


```sql medical_outcomes_all_programs_selected
select
{{program_filter.selected}} as program_selection
```


{% if
    data="medical_outcomes_all_programs_selected"
    condition="has_rows"
    where="program_selection = 'all'"
    %}

    {% stack card=true %}

    {% stack 
        card=false
        width=70
        %}
            
        ### {{$translations.section_descriptions.employee_impact.case_outcomes1.title}}
        <!-- Case Outcomes -->

        <!-- Provides a breakdown of the types of case outcomes. -->
        {{$translations.section_descriptions.employee_impact.case_outcomes1.overview}}

    {% /stack %}

    {% callout
        type="info"
        %}
        {{$translations.metric_definitions.employee_impact.case_outcomes.title}} {{$translations.all_program_restrictions}}
        <!-- is only available for **individual** programs -->
    {% /callout %}

    {% /stack %}

{% /if %}

{% else_if
    data="restriction_calculation_medical_outcomes"
    condition="has_rows"
    where="total_cases is not null"
%}

{% stack card=true %}

    ### {{$translations.section_descriptions.employee_impact.case_outcomes1.title}}
        <!-- Case Outcomes -->

        <!-- Provides a breakdown of the types of case outcomes. -->
        {{$translations.section_descriptions.employee_impact.case_outcomes1.overview}}

    {% row %}

        {% pie_chart
            data="restriction_calculation_medical_outcomes"
            title="{{$translations.metric_definitions.employee_impact.case_outcomes.title}}"
            info="{{$translations.metric_definitions.employee_impact.case_outcomes.description}}"
            info_link="#{{$translations.metric_definitions.employee_impact.case_outcomes.info_link}}"
            info_link_title="{{$translations.read_more}}"
            category="outcome"
            value="cases_outcome"
            value_fmt=""
            width=50
            inner_radius="0"
        /%}

        {% table
        data="restriction_calculation_medical_outcomes"
        width=40
        %}
            {% dimension
            value="outcome"
            title="{{$translations.metric_definitions.employee_impact.case_outcomes.outcome}}"
            /%}

            {% dimension
            value="cases_outcome"
            title="{{$translations.metric_definitions.employee_impact.case_outcomes.cases_outcome}}"
            fmt="num"
            /%}

            {% measure
                value="cases_outcome"
                comparison={
                    compare_vs="target"
                    target="cases_outcome_last_year"
                }
                title=""
                viz="delta"
            /%}


        {% /table %}

    {% /row %}

{% /stack %}

{% /else_if %}

{% else %}

{% stack card=true %}

    {% stack 
        card=false
        width=70
        %}
            
        ### {{$translations.section_descriptions.employee_impact.case_outcomes1.title}}
        <!-- Case Outcomes -->

        <!-- Provides a breakdown of the types of case outcomes. -->
        {{$translations.section_descriptions.employee_impact.case_outcomes1.overview}}

    {% /stack %}

    {% callout
    type="warning"
    %}
    {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.case_outcomes.title}}{{$translations.privacy_disclaimer_end}}
    {% /callout %}

{% /stack %}

{% /else %}
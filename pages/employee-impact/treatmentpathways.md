---
name: treatment_pathways
assetId: 9be8a55e-e4f8-4de3-93b1-188cb3bc8c29
type: partial
---

```sql metric_calculation_treatment_pathways
with case_intensity as (
select case
    when mental_health_treatment_pathway = 'low_intensity' then '{{$translations.metric_definitions.employee_impact.treatment_pathway_breakdown.low_intensity}}'
    when mental_health_treatment_pathway = 'moderate_intensity' then '{{$translations.metric_definitions.employee_impact.treatment_pathway_breakdown.med_intensity}}'
    when mental_health_treatment_pathway = 'high_intensity' then '{{$translations.metric_definitions.employee_impact.treatment_pathway_breakdown.high_intensity}}' 
    end as mental_health_treatment_pathway
    , count(distinct case when date_month_est between {{date_start.selected}} and {{date_end.selected}} then episode_id end) as cases
    , count(distinct case when date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then episode_id end) as cases_last_year
    , sum (cases) over () as total_cases
    , sum (cases_last_year) over () as total_cases_last_year
from episodes_with_contracts
where mental_health_treatment_pathway is not null
    and program in ('mental_health', 'eap', 'icbt')
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
    and date_month_est between date_trunc('month',addMonths({{date_start.selected}}, -12)) and {{date_end.selected}}
    and date_month_est < date_trunc('month',today())
group by 1
)

select *
    , cases / total_cases as percent_cases
    , cases_last_year / total_cases_last_year as percent_cases_last_year
from case_intensity
```

```sql restriction_treatment_pathways
with apply_restrictions as (
select mental_health_treatment_pathway
    , case when cases < {{$privacy_restriction_medium}} then null
    else percent_cases end as percent_cases
    , case when cases < {{$privacy_restriction_medium}} then null
    else cases end as cases
    , case when cases_last_year < {{$privacy_restriction_medium}} then null
    else percent_cases_last_year end as percent_cases_last_year
    , case when cases_last_year < {{$privacy_restriction_medium}} then null
    else cases_last_year end as cases_last_year
    , sum(cases) over () as total_cases
from {{metric_calculation_treatment_pathways}}
)

select *
from apply_restrictions
where cases > 0
```


{% if
    data="restriction_treatment_pathways"
    condition="has_rows"
    where="total_cases is not null"
%}

{% stack card=true %}

    ### {{$translations.section_descriptions.employee_impact.treatment_pathways.title}}
    <!-- Treatment Pathways (Mental Health) -->

    <!-- Provides a breakdown of the levels of mental health interventions members receive based on the intensity of their symptoms. -->
    {{$translations.section_descriptions.employee_impact.treatment_pathways.overview}}


        {% row %}

            {% pie_chart
                data="restriction_treatment_pathways"
                width=50
                category="mental_health_treatment_pathway"
                value="cases"
                value_fmt=""
                inner_radius="0"
                title="{{$translations.metric_definitions.employee_impact.treatment_pathway_breakdown.title}}"
                info="{{$translations.metric_definitions.employee_impact.treatment_pathway_breakdown.description}}"
                info_link="#{{$translations.metric_definitions.employee_impact.treatment_pathway_breakdown.info_link}}"
                info_link_title="{{$translations.read_more}}"
            /%}

            {% table
            data="restriction_treatment_pathways"
            width=40
            order="percent_cases desc"
            %}
                {% dimension
                value="mental_health_treatment_pathway"
                title="{{$translations.metric_definitions.employee_impact.treatment_pathway_breakdown.treatment_path}}"
                /%}

                {% dimension
                value="percent_cases"
                title="{{$translations.metric_definitions.employee_impact.treatment_pathway_breakdown.percent_cases}}"
                fmt="pct"
                /%}

                {% measure
                    value="percent_cases"
                    comparison={
                        compare_vs="target"
                        target="percent_cases_last_year"
                    }
                    title=""
                    viz="delta"
                /%}
        {% /table %}
    
    {% /row %}

{% /stack %}

{% /if %}

{% else %}

    {% stack card=true %}

    {% stack 
        card=false
        width=70
        %}
            
        ### {{$translations.section_descriptions.employee_impact.treatment_pathways.title}}
        <!-- Treatment Pathways (Mental Health) -->

        <!-- Provides a breakdown of the levels of mental health interventions members receive based on the intensity of their symptoms. -->
        {{$translations.section_descriptions.employee_impact.treatment_pathways.overview}}

    {% /stack %}

        {% callout
            type="warning"
            %}
            {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.employee_impact.treatment_pathway_breakdown.title}}{{$translations.privacy_disclaimer_end}}

        {% /callout %}

    {% /stack %}

{% /else %}

---
name: unique_users_comparison
assetId: c1df26ec-2c53-4bf9-aa67-8c0bea3d7d68
type: partial
---

```sql metric_calculation_unique_users_comparision

with program_metrics as (
    select
        case
            when program = 'primary_care' then 
            case when reporting_block_id = 'dia' then '{{$translations.metric_definitions.filter.program_filter.pc.dia }}' 
            when reporting_block_id = 'canadalife' then '{{$translations.metric_definitions.filter.program_filter.pc.canadalife }}'
            when reporting_block_id = 'sunlife' then '{{$translations.metric_definitions.filter.program_filter.pc.sunlife }}'
            when reporting_block_id = 'student-solutions' then '{{$translations.metric_definitions.filter.program_filter.pc.student-solutions }}'
            when reporting_block_id = 'ADP' then '{{$translations.metric_definitions.filter.program_filter.pc.adp }}'
            when reporting_block_id = 'Equitable Life' then '{{$translations.metric_definitions.filter.program_filter.pc.eq_life }}'
            end
            
            when program = 'mental_health' then 
            case 
            when reporting_block_id = 'dia' then '{{$translations.metric_definitions.filter.program_filter.mh.dia }}' 
            when reporting_block_id = 'canadalife' then '{{$translations.metric_definitions.filter.program_filter.mh.canadalife }}'
            when reporting_block_id = 'sunlife' then '{{$translations.metric_definitions.filter.program_filter.mh.sunlife }}'
            when reporting_block_id = 'student-solutions' then '{{$translations.metric_definitions.filter.program_filter.mh.student-solutions }}'
            when reporting_block_id = 'ADP' then '{{$translations.metric_definitions.filter.program_filter.mh.adp }}'
            when reporting_block_id = 'Equitable Life' then '{{$translations.metric_definitions.filter.program_filter.mh.eq_life }}'
            end

            when program = 'eap' then 
            case 
            when reporting_block_id = 'dia' then '{{$translations.metric_definitions.filter.program_filter.eap.dia }}' 
            when reporting_block_id = 'canadalife' then '{{$translations.metric_definitions.filter.program_filter.eap.canadalife }}'
            when reporting_block_id = 'sunlife' then '{{$translations.metric_definitions.filter.program_filter.eap.sunlife }}'
            when reporting_block_id = 'student-solutions' then '{{$translations.metric_definitions.filter.program_filter.eap.student-solutions }}'
            when reporting_block_id = 'ADP' then '{{$translations.metric_definitions.filter.program_filter.eap.adp }}'
            when reporting_block_id = 'Equitable Life' then '{{$translations.metric_definitions.filter.program_filter.eap.eq_life }}'
            end

            when program = 'wellness' then '{{$translations.metric_definitions.filter.program_filter.wellness}}'
            when program = 'icbt' then '{{$translations.metric_definitions.filter.program_filter.icbt}}'
        end as program_temp
        , count(distinct case when date_month between {{date_start.selected}} and {{date_end.selected}} then user_id end) as unique_users
        , count(distinct case when date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and date_trunc('month',addMonths({{date_end.selected}}, -12)) then user_id end) as unique_users_last_year
    from active_users
    where program is not null
        and program not in ('ihp', 'mso')
        and {{filter_value.filter}}
        and reporting_block_id = '{{ $reporting_block_id }}'
        and date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and {{date_end.selected}}
    group by 1
)

, monthly_totals as (
    select date_month 
        , sum(max_registered_employees) as max_registered_employees
    from client_reporting_monthly
    where {{filter_value.filter}}
        and reporting_block_id = '{{ $reporting_block_id }}'
        and date_month between date_trunc('month',addMonths({{date_start.selected}}, -12)) and {{date_end.selected}}
    group by 1
)

, registration_metrics as (
select 
    avg(case when date_month >= {{date_start.selected}}
    and date_month <= {{date_end.selected}} then max_registered_employees end) as avg_registered_employees_current
    , avg(case when date_month >= date_trunc('month',addMonths({{date_start.selected}}, -12))
    and date_month <= date_trunc('month',addMonths({{date_end.selected}}, -12)) then max_registered_employees end) as avg_registered_employees_last_year
    from monthly_totals
)

select 
    program_metrics.program_temp as program,
    program_metrics.unique_users,
    program_metrics.unique_users_last_year as unique_users_previous_year,
    registration_metrics.avg_registered_employees_current,
    registration_metrics.avg_registered_employees_last_year as avg_registered_employees_previous_year,
    divideOrNull(toFloat32(program_metrics.unique_users), toFloat32(registration_metrics.avg_registered_employees_current::float)) as percent_users_current,
    divideOrNull(toFloat32(program_metrics.unique_users_last_year), toFloat32(registration_metrics.avg_registered_employees_last_year)) as percent_users_previous_year
from program_metrics
cross join registration_metrics
```

```sql restriction_calculation_unique_users_comparision
with apply_restrictions as (
select program
    , case when unique_users < {{$privacy_restriction_low}} then null
    else unique_users end as unique_users
    , case when unique_users_previous_year < {{$privacy_restriction_low}} then null
    else unique_users_previous_year end as unique_users_previous_year
    , case when avg_registered_employees_current < {{$privacy_restriction_low}} then null
    else percent_users_current end as percent_users_current
    , case when avg_registered_employees_previous_year < {{$privacy_restriction_low}} then null
    else percent_users_previous_year end as percent_users_previous_year
    , sum(unique_users) over () as total_unique_users
from {{metric_calculation_unique_users_comparision}}
)

select *
from apply_restrictions
where unique_users > 0
```


{% if
    data="restriction_calculation_unique_users_comparision"
    condition="has_rows"
    where="total_unique_users is not null"
    %}

    {% row
        card=true
        align="center"
        %}

        {% pie_chart
            data="restriction_calculation_unique_users_comparision"
            category="program"
            value="unique_users"
            width=30
            inner_radius="0"
        /%}

        {% table
            data="restriction_calculation_unique_users_comparision"
            title="{{$translations.metric_definitions.utilization.unique_user_per_program.title}}"
            info="{{$translations.metric_definitions.utilization.unique_user_per_program.description}}"
            info_link="#{{$translations.metric_definitions.utilization.unique_user_per_program.info_link}}"
            info_link_title="{{ $translations.read_more}}"
            width=60
            %}

            {% dimension
            value="program"
            title="{{$translations.metric_definitions.utilization.unique_user_per_program.program}}"
            /%}

            {% dimension
            value="unique_users"
            title="{{$translations.metric_definitions.utilization.unique_user_per_program.unique_users}}"
            fmt="num"
            /%}

            {% measure
                value="unique_users"
                comparison={
                    compare_vs="target"
                    target="unique_users_previous_year"
                    pct_fmt="pct1"
                }
                title=""
                viz="delta"
            /%}

            {% dimension
            value="percent_users_current"
            title="{{$translations.metric_definitions.utilization.unique_user_per_program.percent_users}}"
            fmt="pct"
            info="{{$translations.metric_definitions.utilization.unique_user_per_program.percent_users_description}}"
            /%}

            {% measure
                value="percent_users_current"
                comparison={
                    compare_vs="target"
                    target="percent_users_previous_year"
                    pct_fmt="pct1"
                }
                title=""
                viz="delta"
            /%}

        {% /table %}

    {% /row %}

{% /if %}

{% else %}

    {% callout
        type="warning"
        %}

        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.unique_user_per_program.title}}{{$translations.privacy_disclaimer_end}}

    {% /callout %}

{% /else %}

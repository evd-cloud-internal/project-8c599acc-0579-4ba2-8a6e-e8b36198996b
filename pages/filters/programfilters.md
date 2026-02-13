---
name: program_filters
assetId: 6b887e8e-699e-4c57-80a8-9604a6dae3b9
type: partial
---

```sql filter_value_program
select 'primary_care' as program, '{{$reporting_block_id}}' as reporting_block_id, 
case 
when reporting_block_id = 'dia' then '{{$translations.metric_definitions.filter.program_filter.pc.dia }}' 
when reporting_block_id = 'canadalife' then '{{$translations.metric_definitions.filter.program_filter.pc.canadalife }}'
when reporting_block_id = 'sunlife' then '{{$translations.metric_definitions.filter.program_filter.pc.sunlife }}'
when reporting_block_id = 'student-solutions' then '{{$translations.metric_definitions.filter.program_filter.pc.student-solutions }}'
when reporting_block_id = 'ADP' then null 
when reporting_block_id = 'Equitable Life' then '{{$translations.metric_definitions.filter.program_filter.pc.eq_life }}' 
end as label
union all
select 'eap' as program, '{{$reporting_block_id}}' as reporting_block_id, 
case 
when reporting_block_id = 'dia' then '{{$translations.metric_definitions.filter.program_filter.eap.dia }}' 
when reporting_block_id = 'canadalife' then '{{$translations.metric_definitions.filter.program_filter.eap.canadalife }}'
when reporting_block_id = 'sunlife' then '{{$translations.metric_definitions.filter.program_filter.eap.sunlife }}'
when reporting_block_id = 'student-solutions' then '{{$translations.metric_definitions.filter.program_filter.eap.student-solutions }}'
when reporting_block_id = 'ADP' then '{{$translations.metric_definitions.filter.program_filter.eap.adp }}' 
when reporting_block_id = 'Equitable Life' then null
end as label
union all
select 'mental_health' as program, '{{$reporting_block_id}}' as reporting_block_id, 
case 
when reporting_block_id = 'dia' then '{{$translations.metric_definitions.filter.program_filter.mh.dia }}' 
when reporting_block_id = 'canadalife' then '{{$translations.metric_definitions.filter.program_filter.mh.canadalife }}'
when reporting_block_id = 'sunlife' then '{{$translations.metric_definitions.filter.program_filter.mh.sunlife }}'
when reporting_block_id = 'student-solutions' then '{{$translations.metric_definitions.filter.program_filter.mh.student-solutions }}'
when reporting_block_id = 'ADP' then null
when reporting_block_id = 'Equitable Life' then null
end as label
union all
select 'all' as program, '{{$reporting_block_id}}' as reporting_block_id, 
case
when reporting_block_id = 'ADP' then null
when reporting_block_id = 'Equitable Life' then null
else '{{ $translations.metric_definitions.filter.program_filter.all}}' end as label
```

<!-- {% dropdown
    id="program_filter"
    data="filter_value_program"
    value_column="program"
    initial_value="primary_care"
    label_column="label"
    info="Select which dialogue program to filter on"
/%} -->

{% dropdown
    id="program_filter"
    title="{{ $translations.metric_definitions.filter.program_filter.title}}"
    data="filter_value_program"
    value_column="program"
    initial_value="primary_care"
    label_column="label"
    info="{{ $translations.metric_definitions.filter.program_filter.description}}"
    where="label is not null"
    select_first=true
/%}
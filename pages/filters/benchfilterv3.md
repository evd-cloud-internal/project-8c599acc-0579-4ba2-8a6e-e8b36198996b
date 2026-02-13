---
name: bench_filter_v3
assetId: 639b6b1c-0d44-4a75-ab34-2b25858560fd
type: partial
---

```sql comparison_select
select 
'Time Comparison (' || formatDateTime(date_trunc('month',addMonths( {{date_start.selected}}, -12 ) ), '%Y-%m') || ' > ' || formatDateTime(date_trunc('month',addMonths( {{date_end.selected}}, -12 ) ), '%Y-%m') || ')' as comparison_type
, '{{ $translations.metric_definitions.filter.benchmark_comparison_filter.time_comparison}} (' || formatDateTime(date_trunc('month',addMonths( {{date_start.selected}}, -12 ) ), '%Y-%m') || ' > ' || formatDateTime(date_trunc('month',addMonths( {{date_end.selected}}, -12 ) ), '%Y-%m') || ')' as label

union all

select 
'Industry' as comparison_type
, '{{ $translations.metric_definitions.filter.benchmark_comparison_filter.industry}}' as label
where {{ $include_industry_benchmark }}

union all

select 
'Similar Organizations' as comparison_type
, '{{ $translations.metric_definitions.filter.benchmark_comparison_filter.similar_org}}' as label
union all

select 
'Block of Business' as comparison_type
, '{{ $translations.metric_definitions.filter.benchmark_comparison_filter.block}}' as label
where '{{$reporting_block_id}}' not in ('dia', 'ADP', 'Equitable Life')
and {{ $include_block_benchmark }}
```


{% dropdown
    id="comparison_filter"
    data="comparison_select"
    title="{{ $translations.metric_definitions.filter.benchmark_comparison_filter.title}}"
    value_column="comparison_type"
    label_column="label"
    select_first=true
    order="comparison_type desc"
    placeholder="Select Comparison type"
    info="{{ $translations.metric_definitions.filter.benchmark_comparison_filter.description}}"
    info_link="#{{ $translations.metric_definitions.filter.benchmark_comparison_filter.info_link}}"
    info_link_title="{{$translations.read_more}}"
/%}


---
name: bench_or_time_comp_filter
assetId: d1754d3d-35e7-4f25-8674-8ac40e8a2f60
type: partial
---

```sql comparison_type_select
select 
'Benchmark' as comparison_type

union all

select 
'Time Comparison' as comparison_type


```
    
{% dropdown
    id="comparison_type_filter"
    data="comparison_type_select"
    value_column="comparison_type"
    initial_value="Benchmark"
    info="Specify whether to compare registration data against a group benchmark or previous year time comparison"
    width=35
/%}





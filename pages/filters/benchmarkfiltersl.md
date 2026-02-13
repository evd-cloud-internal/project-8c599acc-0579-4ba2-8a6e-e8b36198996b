---
name: benchmark_filter_sl
assetId: 3c142c3b-62f5-4df1-8780-fcb2ab1590ed
type: partial
---

```sql benchmark_select
select 
'Similar Organizations' as benchmark_type

union all

select 
'None' as benchmark_type
```

{% dropdown
id="benchmark_filter"
data="benchmark_select"
value_column="benchmark_type"
placeholder="Select benchmark type"
initial_value="Similar Organizations"
info="Similar Organizations benchmark includes other organizations within the same range of  contract-to-date ages and number of members as the selected organization."
/%}


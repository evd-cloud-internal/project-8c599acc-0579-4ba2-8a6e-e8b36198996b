---
name: date_filters
assetId: d8d3b4fd-43fa-43e2-a6b2-385bae01f1bc
type: partial
---

```sql date_spine_selected
select 
cast(toDate(date_month) AS String) as str_date_month
, cast(toDate(month_end) AS String) as str_month_end
from dimension_months
where toDate(date_month) < date_trunc('month', today())
```

{% dropdown
id="date_start"
data="date_spine_selected"
title="{{ $translations.metric_definitions.filter.date_start_filter.title}}"
value_column="str_date_month"
order="str_date_month desc"
placeholder="Select start date"
initial_value="2025-01-01"
info="{{ $translations.metric_definitions.filter.date_start_filter.description}}"
/%}

{% dropdown
id="date_end"
data="date_spine_selected"
title="{{ $translations.metric_definitions.filter.date_end_filter.title}}"
value_column="str_month_end"
order="str_month_end desc"
placeholder="Select end date"
select_first=true
info="{{ $translations.metric_definitions.filter.date_end_filter.description}}"
/%}

```sql interval_granularity_deprecated
select 'YEAR' as granularity, '12 months' as label
union all
select 'QUARTER' as granularity, '3 months' as label
union all
select 'MONTH' as granularity, '1 month' as label
```

```sql period_comparison
select '12' as num, 'Year to year' as label
```
<!-- union all
select cast(date_diff('month', toDate({{date_start.selected}}), toDate({{date_end.selected}})) + 1 AS String) as num, 'Period to period' as label -->
<!-- {% dropdown
    id="interval_granularity"
    data="period_comparison"
    value_column="num"
    label_column="label"
    initial_value="12"
    title="Time Comparison"
    placeholder="Select type"
/%} -->

```sql number_of_intervals
select '1' as intervals
union all
select '2' as intervals
union all
select '3' as intervals
```
<!-- {% dropdown
    id="number_of_previous_periods"
    data="number_of_intervals"
    value_column="intervals"
    initial_value="1"
/%} -->
---
name: member_type_filters
assetId: a62d2f7c-5914-4c11-91fe-efafaa40e03c
type: partial
---

```sql member_type_select
select 
'Family' as member_type
, '{{ $translations.metric_definitions.filter.member_type_filter.family}}' as label

union all

select 
'Member' as member_type
, '{{ $translations.metric_definitions.filter.member_type_filter.employee}}' as label
union all

select 
'All' as member_type
, '{{ $translations.metric_definitions.filter.member_type_filter.all}}' as label
```

{% dropdown
id="member_type_filter"
data="member_type_select"
title="{{ $translations.metric_definitions.filter.member_type_filter.title}}"
value_column="member_type"
placeholder="Select member type"
label_column="label"
initial_value="All"
info="{{ $translations.metric_definitions.filter.member_type_filter.description}}"
/%}


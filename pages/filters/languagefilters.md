---
name: language_filters
assetId: 2fe7c2df-4f36-4e3d-a739-e47aeb5af87e
type: partial
---

```sql language_filter_table
select 'EN' as value, 'English' as label
union all
select 'FR' as value, 'Français' as label
```
{% dropdown
    id="language_filter"
    data="language_filter_table"
    value_column="value"
    label_column="label"
    initial_value="EN"
/%}

<!-- ```sql language_filter_table
select
    value as description,
    metric_name,
    language 
from base_client_reporting_metric_descriptions
```
{% table_filter
    id="language_filter"
    data="language_filter_table"
    columns=["language"]
/%} -->
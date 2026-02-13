---
name: sl_bdr_filter_set
assetId: b446c3cd-d54e-4a22-b891-45f158dcb04e
type: partial
---

---
reporting_block_id: sunlife
---

```sql filter_tables
select
    sisense_organization_name as organization_name
    , is_employee
    , reporting_column_one 
    , reporting_column_two
    , account_name
    , account_id
from evidence_client_report_account_filters
where reporting_block_id = '{{ $reporting_block_id }}'
```

{% table_filter
    id="filter_value"
    data="filter_tables"
    columns=["organization_name", "is_employee", "reporting_column_one", "reporting_column_two"]
    
    labels=["{{ $translations.metric_definitions.filter.table_filter.organization}}", "{{ $translations.metric_definitions.filter.table_filter.is_employee}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_one_sl}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_two_sl}}"]
    multiple=false
    initial_values={
        organization_name="025320 - Honda Canada Inc"
    }
    require_selection=["organization_name"]
/%}

```sql filtered_table
select *
from {{filter_tables}}
where {{filter_value.filter}}
```
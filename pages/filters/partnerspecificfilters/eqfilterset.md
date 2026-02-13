---
name: eq_filter_set
assetId: 1b107685-520d-4eba-a626-2f12ddae751e
type: partial
---

---
reporting_block_id: Equitable Life
---

```sql filter_tables
select
    sisense_organization_name as organization_name
    , billing_administration
    , broker_advisor_name
    , insurance_carrier_name 
    , reporting_block_id
    , reporting_group_id
    , organization_id
    , plan_name 
    , account_name
    , account_id
    , industry
    , benchmark_group
    , reporting_group_id
    , reporting_column_one 
    , reporting_column_two 
    , reporting_column_three 
    , is_employee
from evidence_client_report_account_filters
where reporting_block_id = '{{ $reporting_block_id }}'
```

{% table_filter
    id="filter_value"
    data="filter_tables"
    columns=["account_name", "organization_name", "reporting_column_one", "is_employee"]
    labels=["{{ $translations.metric_definitions.filter.table_filter.account}}", "{{ $translations.metric_definitions.filter.table_filter.organization}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_one_eq}}", "{{ $translations.metric_definitions.filter.table_filter.is_employee}}"]
/%}


```sql filtered_table
select *
from {{filter_tables}}
where {{filter_value.filter}}
```

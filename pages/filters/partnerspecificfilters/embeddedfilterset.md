---
name: embedded_filter_set
assetId: b1a933f0-ff65-4a62-ba9d-4b75342d11b4
type: partial
---

---
reporting_block_id: dia
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
    columns=["organization_name"]
    labels=["{{ $translations.metric_definitions.filter.table_filter.organization}}"]
/%}

```sql filtered_table
select *
from {{filter_tables}}
where {{filter_value.filter}}
```

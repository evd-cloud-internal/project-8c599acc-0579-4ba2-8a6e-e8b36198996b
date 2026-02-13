---
name: dia_filter_set
assetId: 8f505e7d-679d-4fa1-81bb-d439368afad6
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
    columns=["account_name", "organization_name", "reporting_column_one", "reporting_column_two", "reporting_column_three", "is_employee", "insurance_carrier_name", "broker_advisor_name", "billing_administration"]
    labels=["{{ $translations.metric_definitions.filter.table_filter.account}}", "{{ $translations.metric_definitions.filter.table_filter.organization}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_one}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_two}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_three}}", "{{ $translations.metric_definitions.filter.table_filter.is_employee}}", "{{ $translations.metric_definitions.filter.table_filter.insurance_carrier}}", "{{ $translations.metric_definitions.filter.table_filter.broker_advisor}}", "{{ $translations.metric_definitions.filter.table_filter.billing_administration}}"]
/%}


```sql filtered_table
select *
from {{filter_tables}}
where {{filter_value.filter}}
```


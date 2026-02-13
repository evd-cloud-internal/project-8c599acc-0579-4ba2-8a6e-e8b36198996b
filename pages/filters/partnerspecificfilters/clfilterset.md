---
name: cl_filter_set
assetId: be7cbb24-2b78-4c44-9763-e4ed7d8b487d
type: partial
---

---
reporting_block_id: canadalife
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
    , canadalife_client_id
    , canadalife_policy_name 
    , canadalife_policy_number
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
    columns=["account_name", "organization_name", "reporting_column_one", "reporting_column_two", "reporting_column_three", "is_employee", "canadalife_client_id", "canadalife_policy_name", "canadalife_policy_number"]
    labels=["{{ $translations.metric_definitions.filter.table_filter.account}}", "{{ $translations.metric_definitions.filter.table_filter.organization}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_one_cl}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_two_cl}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_three_cl}}", "{{ $translations.metric_definitions.filter.table_filter.is_employee}}", "{{ $translations.metric_definitions.filter.table_filter.canadalife_client_id}}", "{{ $translations.metric_definitions.filter.table_filter.canadalife_policy_name}}", "{{ $translations.metric_definitions.filter.table_filter.canadalife_policy_number}}"]
/%}
 <!-- "client_id", "policy_name", "policy_number" -->
```sql filtered_table
select *
from {{filter_tables}}
where {{filter_value.filter}}
```

---
name: sl_product_filter_set
assetId: 5f85934c-cbad-4fc7-b7c7-de1ffdb2ba80
type: partial
---

---
reporting_block_id: sunlife
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
    , sunlife_business_unit_{{ $translations.inline_query }}
from evidence_client_report_account_filters
where reporting_block_id = '{{ $reporting_block_id }}'
```

{% table_filter
    id="filter_value"
    data="filter_tables"
    columns=["account_name", "organization_name", "reporting_column_one", "reporting_column_two", "reporting_column_three", "is_employee", "sunlife_business_unit_{{ $translations.inline_query }}"]
    labels=["{{ $translations.metric_definitions.filter.table_filter.account}}", "{{ $translations.metric_definitions.filter.table_filter.organization}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_one_sl}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_two_sl}}", "{{ $translations.metric_definitions.filter.table_filter.reporting_column_three}}", "{{ $translations.metric_definitions.filter.table_filter.is_employee}}", "{{ $translations.metric_definitions.filter.table_filter.sunlife_business_unit}}"]
/%}

```sql filtered_table
select *
from {{filter_tables}}
where {{filter_value.filter}}
```


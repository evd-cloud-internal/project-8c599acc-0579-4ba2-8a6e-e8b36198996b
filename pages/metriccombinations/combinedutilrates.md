---
name: combined_util_rates
assetId: 51a2af38-948e-4078-b98a-85d9e6c1b2ee
type: partial
---

```sql is_one_account_annualized_and_regular_utilization_rate
select
count(distinct account_name) as account_count
from {{filtered_table}} as filtered_table
having account_count = 1
```

{% if
    data="is_one_account_annualized_and_regular_utilization_rate"
    condition="has_rows"
    %}

    #### {{$translations.metric_definitions.utilization.utilization_rate.title}}
    <!-- Utilization Rate -->
    
    {% row 
        align="bottom"
        %}
        
        {% partial file="utilization/utilizationratecomparison"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low 
                privacy_restriction_high = $privacy_restriction_high  
            } 
        /%}

        <!-- {% partial file="utilization/annualizedutilizationrate" 
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low 
                    privacy_restriction_high = $privacy_restriction_high      
                }  
        /%} -->

        {% partial file="utilization/utilratetwelvemonthsimple"
                variables={reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low 
                privacy_restriction_high = $privacy_restriction_high                  
            }  
        /%}

    {% /row %}

    {% line_break lines=1 /%}

{% /if %}

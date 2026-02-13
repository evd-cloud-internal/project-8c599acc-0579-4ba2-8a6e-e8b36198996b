---
name: icbt_section
assetId: 3449853d-9126-43a1-b9d0-e6b4e7d2d3b2
type: partial
---

<!-- icbt section is moved into a partial because this should be hidden for certain reports -->
<!-- Partial requires reporting_block_id, privacy_restriction_low vars -->
    
```sql reporting_block_id_check_icbt
select
 1 as check_value
where '{{ $reporting_block_id }}' not in ('ADP', 'Equitable Life')
```
    

{% if
    data="reporting_block_id_check_icbt"
    condition="has_rows"
    %}
    
    {% line_break lines=1 /%}

    # {{$translations.section_descriptions.utilization.icbt.title}}
    <!-- iCBT -->
    <!-- Indicates how members engage with our self-paced interactive iCBT toolkits. -->
    {{$translations.section_descriptions.utilization.icbt.overview}}
    
    {% partial file="utilization/icbtcompletionrate"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low 
            } 
    /%}
    
    {% line_break lines=1 /%}

    {% partial
        file="utilization/icbtmodulescompleted"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low     
        } 
    /%}

{% /if %}
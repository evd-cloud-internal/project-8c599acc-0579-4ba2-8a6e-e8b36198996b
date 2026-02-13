---
name: wellness_section
assetId: 0bd1a8eb-5177-4b84-a11a-b3a08ff98f3a
type: partial
---

<!-- wellness section is moved into a partial because this should be hidden for certain reports -->
<!-- Partial requires reporting_block_id, privacy_restriction_low, privacy_restriction_medium vars -->
    
```sql reporting_block_id_check_wellness
select
 1 as check_value
where '{{ $reporting_block_id }}' not in ('canadalife', 'ADP', 'Equitable Life')
```
    

{% if
    data="reporting_block_id_check_wellness"
    condition="has_rows"
    %}

    {% line_break lines=1 /%}

    # {{$translations.section_descriptions.utilization.wellness.title}}
    <!-- Wellness -->
    <!-- Indicates how members engage with our Wellness content, Healthy Habits, and activities that support preventive health. -->
    {{$translations.section_descriptions.utilization.wellness.overview}}

    ### {{$translations.key_metrics}}
    {% row  %}
    
        {% partial file="utilization/adoptionrate"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low 
            }  
        /%}
        
        {% partial file="utilization/challenges-reach"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low 
            }  
        /%}

    {% /row %}
    
    {% line_break lines=1 /%}
    
    {% partial file="utilization/tophabitscollection"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_medium = $privacy_restriction_medium    
        } 
    /%}

    {% line_break lines=1 /%}

    {% partial file="utilization/dailystepsweeklyactiveminutes"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low     
        } 
    /%}

    {% line_break lines=1 /%}

{% /if %}
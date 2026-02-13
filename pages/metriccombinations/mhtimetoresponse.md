---
name: mh_time_to_response
assetId: 85426beb-3676-4595-bdd6-142e5b2c6f6d
type: partial
---

{% stack card=true %}    
   
    ### {{$translations.section_descriptions.employee_impact.mental_health.title}}
    <!-- Mental Health -->
    <!-- Focuses specifically on the time it takes for mental health symptoms to improve following treatment. -->
    {{$translations.section_descriptions.employee_impact.mental_health.overview}}
    
    {% row 
        align="stretch"
        card=true
        %}

        {% partial file="employee-impact/response-to-therapy-anxiety"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_medium = $privacy_restriction_medium
            }
        /%}

        {% partial file="employee-impact/response-to-therapy-depression"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_medium = $privacy_restriction_medium
            }
        /%}
    
    {% /row %}

{% /stack %}

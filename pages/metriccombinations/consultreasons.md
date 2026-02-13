---
name: consult_reasons
assetId: 6cf4c28d-6d78-4b7d-91ce-940aebb59791
type: partial
---

---
reporting_block_id: dia
---

{% stack card=true %}

    ### {{$translations.section_descriptions.utilization.consult_reasons.title}}
    <!-- Consult Reasons -->
    <!-- Indicates the most recurring needs and concerns that drive members to seek support. -->
    {{$translations.section_descriptions.utilization.consult_reasons.overview}}


    {% partial file="utilization/topreasonforconsult" 
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_medium = $privacy_restriction_medium
        }  
    /%}

    {% line_break lines=1 /%}

    {% partial file="utilization/mhreasonsconsultationcomparison" 
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_medium = $privacy_restriction_medium
        }  
    /%}

{% /stack %}

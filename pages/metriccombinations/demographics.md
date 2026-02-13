---
name: demographics
assetId: 8121f925-3817-462e-8637-44a415a6a19a
type: partial
---

{% stack card=true %}

    ### {{$translations.section_descriptions.utilization.demographics.title}}
    <!-- Demographics  -->
    <!-- Shows which demographics are the most vs least engaged. -->
    {{$translations.section_descriptions.utilization.demographics.overview}}

    {% partial file="utilization/memberprovince" 
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_demographics_bucket = $privacy_restriction_demographics_bucket
            privacy_restriction_demographics_members = $privacy_restriction_demographics_members
        }  
    /%}

    {% partial file="utilization/agegroup"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_demographics_bucket = $privacy_restriction_demographics_bucket
            privacy_restriction_demographics_members = $privacy_restriction_demographics_members
        }   
    /%}

{% /stack %}




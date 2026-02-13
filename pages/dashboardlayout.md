---
name: dashboard_layout
assetId: 1135c227-4c2e-4881-b43e-c392fcb21d3c
type: partial
---

---
privacy_restriction_lowest: 3
privacy_restriction_low: 5
privacy_restriction_medium: 10
privacy_restriction_high: 35
privacy_restriction_demographics_bucket: 20
privacy_restriction_demographics_members: 100
include_block_benchmark: true
include_industry_benchmark: true
---
{% tabs full_width=true %}

<!-- Utilization Tab -->
{% tab 
    title="{{$translations.section_descriptions.utilization.utilization.title}}" 
    icon="eye"
    print_break="auto"
    %}

    <!-- # Utilization -->
    {{$translations.section_descriptions.utilization.utilization.overview}}

    {% line_break lines=1 /%}

        # {{$translations.section_descriptions.utilization.registration.title}}
        {{$translations.section_descriptions.utilization.registration.overview}}
    
    {% row card=false %}

        {% stack 
            card=false 
            width=70
            %}

            ### {{$translations.key_metrics}}

        {% /stack %}
        
        {% partial file="filters/benchfilterv3"
            variables={
                reporting_block_id = $reporting_block_id
                include_block_benchmark = $include_block_benchmark
                include_industry_benchmark = $include_industry_benchmark
            }  
        /%}

    {% /row %}

    {% line_break lines=1 /%}

    {% row%}

        {% partial file="utilization/registrationratecomparison" 
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_lowest = $privacy_restriction_lowest
                privacy_restriction_low = $privacy_restriction_low
                privacy_restriction_high = $privacy_restriction_high
                privacy_restriction_demographics_bucket = $privacy_restriction_demographics_bucket
            }  
        /%}

        {% partial file="utilization/activationratecomparison" 
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_lowest = $privacy_restriction_lowest
                privacy_restriction_low = $privacy_restriction_low                
                privacy_restriction_demographics_bucket = $privacy_restriction_demographics_bucket
            }  
        /%}

    {% /row %}

    {% line_break lines=1 /%}

    {% partial file="utilization/regfunnel"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low
            privacy_restriction_high = $privacy_restriction_high
        }  
    /%}
    
    {% line_break lines=1 /%}
    
    {% partial file="utilization/memberfunnelovertime"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low
            privacy_restriction_high = $privacy_restriction_high
        }  
    /%}

    {% line_break lines=1 /%}

    {% partial file="metriccombinations/demographics"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_demographics_bucket = $privacy_restriction_demographics_bucket
            privacy_restriction_demographics_members = $privacy_restriction_demographics_members
        } 
    /%}

    # {{$translations.section_descriptions.utilization.engagement_1.title}}
    <!-- Engagement  -->
    
    <!-- Indicates how frequently members return, explore, or engage with core features, revealing overall adoption health. -->

    {{$translations.section_descriptions.utilization.engagement_1.overview}}

    {% row card=false %}
    
        ### {{$translations.key_metrics}}

        {% partial file="filters/benchfilterv3" 
            variables={
                reporting_block_id = $reporting_block_id
                include_block_benchmark = $include_block_benchmark
                include_industry_benchmark = $include_industry_benchmark
            }
        /%}

    {% /row %}

    {% partial file="utilization/engagementratecomparison"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_lowest = $privacy_restriction_lowest
            privacy_restriction_low = $privacy_restriction_low
            privacy_restriction_demographics_bucket = $privacy_restriction_demographics_bucket
        } 
    /%}

    {% line_break lines=1 /%}
            
    {% partial file="utilization/monthlyengagementrate"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low
        }  
    /%}

    {% line_break lines=1 /%}

    {% partial file="utilization/uniqueuserscomparison"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low
        }  
    /%}

    {% line_break lines=1 /%}

    {% stack
        card=true
        %}

        {% row card=false %}

            {% stack 
                card=false
                width=70
                %}
                # {{$translations.section_descriptions.utilization.program.title}}
                <!-- Program -->
                <!-- Indicates how frequently members use Dialogue's services at the program level.    -->
                {{$translations.section_descriptions.utilization.program.overview}}
            {% /stack %}

            {% partial file="filters/programfilters"
                variables={
                reporting_block_id = $reporting_block_id
                } 
            /%}

        {% /row %}

        {% line_break lines=1 /%}

        ### {{$translations.key_metrics}}
        <!-- Shows how many employees are using each program relative to eligiblity, indicating reach and relevance. -->

        {% row card=false %}
        
            {% partial file="utilization/totalsessionsbigvalue"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low    
                }    
            /%}

            {% partial file="utilization/totalcasesbigvalue"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low  
                }   
            /%}

            {% partial file="utilization/consultationpercase"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low      
                }  
            /%}
        
        {% /row %}

        {% line_break lines=1 /%}


        {% stack card=true %}

            {% partial file="metriccombinations/combinedutilrates"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low 
                    privacy_restriction_high = $privacy_restriction_high 
                }
            /%}
            <!-- this partial combines ctd, 12 months, annualized util rate metrics -->
            <!-- this is done because card has to be hidden when multiple account selected -->

            {% partial file="utilization/monthlyutilizationrateyoy" 
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low     
                }  
            /%}

        {% /stack %}

        {% line_break lines=1 /%}

        {% partial file="utilization/totalsessionsmonthly"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low   
                }    
        /%}
            
        {% line_break lines=1 /%}

        {% partial file="utilization/totalcasesmonthly"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low     
            }   
        /%}

        {% line_break lines=1 /%}

        {% partial file="metriccombinations/consultreasons"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_medium = $privacy_restriction_medium
            }  
        /%}
        
    {% /stack %}

    <!-- this partial contains entire icbt section -->
    {% partial
        file="metriccombinations/icbtsection"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low             
        }
    /%}
    
    <!-- this partial contains entire wellness section -->
    {% partial
        file="metriccombinations/wellnesssection"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low 
            privacy_restriction_medium = $privacy_restriction_medium
        }
    /%}
    
    # {{$translations.glossary}}
    <!-- {% details
        title="Glossary"
        open=true
    %} -->
    {% partial
        file="glossary/metricutilization"
        variables={
            metric_name_array = "('Cases', 'Sessions', 'Consultations', 'Episodes','Registration Rate', 'Activation Rate', 'Total Sessions', 'Total Cases', 'Unique Users per Program', 'Activation Funnel Over Time', 'Top Consultation Reasons by Program', 'Reasons for Mental Health Consultations', 'Monthly Utilization Rate', 'Monthly Engagement Rate', 'Annualized Utilization Rate', '12-month Utilization Rate', 'Contract-to-date Utilization Rate', 'Challenges Reach', 'Adoption Rate', 'Top Habit Collections', 'iCBT Completion Rate', 'Member Province', 'Age Group', 'Consultations per Case', 'Engagement Rate', 'Activation Funnel','Utilization Rate', 'Daily Steps & Weekly Active Minutes', 'Top Toolkits', 'Benchmark comparisons', 'Monthly Cases', 'Monthly Sessions')"
        }
    /%}
        <!-- {% /details %} -->

{% /tab %}


<!-- Satisfaction Tab -->
{% tab 
    title="{{$translations.section_descriptions.satisfaction.satisfaction.title}}" 
    icon="heart"
    %}

    <!-- Indicates levels of overall satisfaction, showing how members rate their experiences across different services. -->
    {{$translations.section_descriptions.satisfaction.satisfaction.overview}}
    {% stack card=true %}

        {% row card=false %}

            {% stack 
                card=false 
                width=70
                %}
            
                ## {{$translations.section_descriptions.satisfaction.post_appointment_survey.title}}
                <!-- Post-Appointment Survey -->
                <!-- Indicates general sentiment, confidence, and percieved value of care. -->
                {{$translations.section_descriptions.satisfaction.post_appointment_survey.overview}}

            {% /stack %}

            {% partial file="filters/programfilters"
                variables={
                reporting_block_id = $reporting_block_id
                } 
            /%}

        {% /row %}

        ### {{$translations.key_metrics}}

        {% row 
            card=false 
            align="bottom"
            %}

            {% partial file="satisfaction/avgsatisfactionscore"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low      
                }
            /%}

            {% partial file="satisfaction/netpromoterscore"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low       
                }
            /%}

            {% partial file="satisfaction/numberofsatisfactionsurveys" 
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low      
                }
            /%}

        {% /row %}

        {% line_break lines=1 /%}

        {% partial file="satisfaction/satisfactionovertime"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low    
            }
        /%}

        {% line_break lines=1 /%}

        {% partial file="satisfaction/npsovertime"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low    
            }
        /%}
        
    {% /stack %}

    {% line_break lines=1 /%}

    ## What Members are saying about our services

    find out what youre members are thinking, in their own words

    {% line_break lines=1 /%}

    {% partial file="satisfaction/aiexecsummary" /%}

    {% line_break lines=1 /%}

    {% partial file="satisfaction/aimetricbucketspositive"/%}

    {% line_break lines=1 /%}

    {% partial file="satisfaction/aimetricbucketsnegative" /%}
    
    # {{$translations.glossary}}
    <!-- {% details
        title="Satisfaction Glossary"
        open=true
    %} -->
            {% partial
            file="glossary/metricsatisfaction"
            variables={
                metric_name_array = "('Net Promoter Score (NPS)', 'Average Satisfaction Score', 'Number of Satisfaction Surveys', )"
            }
        /%}
    <!-- {% /details %} -->
{% /tab %}

<!-- Employee Impact Tab -->
{% tab
    title="{{$translations.section_descriptions.employee_impact.employee_impact.title}}"
    icon="trending-up"
    %}

    # {{$translations.section_descriptions.employee_impact.productivity.title}}
    <!-- Productivity -->
    <!-- Indicates how timely, effective support helps members recover faster, miss fewer days, and maintain better focus at work. -->
    {{$translations.section_descriptions.employee_impact.productivity.overview}}
    

    ## {{$translations.section_descriptions.employee_impact.time_saved.title}}
    <!-- Time Saved -->
    <!-- Shows how efficient care and early intervention result in time savings for members. -->
    {{$translations.section_descriptions.employee_impact.time_saved.overview}}

    {% row
        card=false
        %}

            {% partial file="employee-impact/timesavedavg"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low      
                }
            /%}

            {% partial file="employee-impact/timesavedsurveycount"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low     
                }
            /%}

            {% partial file="employee-impact/timesavedparticipationrate"
                variables={
                    reporting_block_id = $reporting_block_id
                    privacy_restriction_low = $privacy_restriction_low    
                }
            /%}

    {% /row %}

    {% line_break lines=1 /%}

    # {{$translations.section_descriptions.employee_impact.outcomes.title}}
    <!-- Outcomes -->
    <!-- Demonstrates the real impact of Dialogue's programs on members' physical and mental well-being over time. -->
    {{$translations.section_descriptions.employee_impact.outcomes.overview}}

    ## {{$translations.section_descriptions.employee_impact.wellbeing.title}}
    <!-- Well-being -->
    <!-- Indicates how members feel day-to-day across physical, mental, and emotional dimensions. -->
    {{$translations.section_descriptions.employee_impact.wellbeing.overview}}

    ### {{$translations.key_metrics}}

    {% row align="stretch"%}
    
        {% partial file="employee-impact/wbs-score"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low
            }
        /%}

        {% partial file="employee-impact/wbs-survey-count"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low
            }
        /%}

        {% partial file="employee-impact/wbs-participation-rate"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low
            }
        /%}

    {% /row %}

    {% line_break lines=1 /%}

    {% partial file="employee-impact/wbs-score-over-time"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low
        }
    /%}

    {% line_break lines=1 /%}

    {% partial file="employee-impact/wbs-focus-areas"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_low = $privacy_restriction_low
        }
    /%}

    {% line_break lines=1 /%}

    ## {{$translations.section_descriptions.employee_impact.case_outcomes.title}}
    <!-- Case Outcomes  -->
    <!-- Shows how different types of cases unfold across outcomes and treatment pathways for physical and mental health. -->
    {{$translations.section_descriptions.employee_impact.case_outcomes.overview}}


    {% partial file="employee-impact/medical-outcomes"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_low = $privacy_restriction_low
                }
    /%}

    {% line_break lines=1 /%}

    {% partial file="employee-impact/treatmentpathways"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_medium = $privacy_restriction_medium
        }
    /%}

    {% line_break lines=1 /%}

    ## {{$translations.section_descriptions.employee_impact.health_outcomes.title}}
    <!-- Health Outcomes -->
    <!-- Indicates how often care leads to meaningful recovery or improvement in members' conditions. -->
    {{$translations.section_descriptions.employee_impact.health_outcomes.overview}}

    {% line_break lines=1 /%}

    ### {{$translations.key_metrics}}

    <!-- {% row 
        align="stretch"
        %} -->

        {% partial file="employee-impact/effectiveness-of-care"
            variables={
                reporting_block_id = $reporting_block_id
                privacy_restriction_medium = $privacy_restriction_medium
            }
        /%}
    <!-- {% /row %} -->

    {% line_break lines=1 /%}

    ## {{$translations.section_descriptions.employee_impact.time_to_improvement.title}}
    <!-- Time to Improvement -->
    <!-- Indicates how soon members begin to feel better after seeking care. -->
    {{$translations.section_descriptions.employee_impact.time_to_improvement.overview}}

    {% partial file="metriccombinations/mhtimetoresponse"
        variables={
            reporting_block_id = $reporting_block_id
            privacy_restriction_medium = $privacy_restriction_medium
        }
    /%}

    {% line_break lines=1 /%}

    # {{$translations.glossary}}
    <!-- {% details
        title="Employee Impact Glossary"
        open=true
    %} -->
    {% partial
        file="glossary/metricemployeeimpact"
        variables={
            metric_name_array = "('Average Time Saved', 'Time to Response to Therapy (Anxiety)', 'Time to Response To Therapy (Depression)', 'Case Outcomes', 'Member Well-Being Focus Areas', 'Effectiveness of Care (Overall)', 'Treatment Pathways Breakdown', 'Average Well-Being Score Overtime', 'Average Well-Being Score', 'Number of Time Saved Surveys', 'Number of WBS Surveys', 'Participation Rate of WBS Surveys', 'Participation Rate of Time Saved Surveys')"
        }
    /%}
    <!-- {% /details %} -->


{% /tab %}

{% /tabs %}
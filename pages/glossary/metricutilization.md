---
name: metric_utilization
assetId: 1df321eb-1612-4e70-89c7-051b1fb96ba4
type: partial
---

<!-- ---
metric_name_array: ('Registration Rate', 'Activation Rate', 'Total Sessions', 'Total Cases', 'Unique Users per Program', 'Member Funnel Overtime', 'Top Reasons for Consult', 'Reasons for Mental Health Consultations', 'Total Cases (exec summary)', 'Monthly Utilization Rate', 'Monthly Engagement Rate', 'Treatment Pathways Breakdown', 'Annualized Utilization Rate', '12-month Utilization Rate', 'Contract-to-date Utilization Rate', 'Challenges Reach', 'Adoption Rate', 'Top Habit Collections', 'iCBT Completion Rate', 'Member Province', 'Age Group', 'Daily Steps & Weekly Active Minutes', 'Top Toolkits')
--- -->
```sql metric_utilization
select * 
from base_client_reporting_metric_descriptions 
where lower(language) = '{{$translations.inline_query}}' and metric_name in {{$metric_name_array}}
order by metric_name desc

union all

select 'vs. last year' as metric_name, 'vs. last year' as display_name, 'en' as language, 'All year-over-year comparisons compare against the same selected period 12 months ago. Here the selected period starts from ' || {{date_start}} || ' to ' || {{date_end}} || ', vs. last year compares against period from ' || toDate(addMonths({{date_start}}, -12)) || ' to ' ||  toDate(addMonths({{date_end}}, -12)) as value

union all

select 'vs. last year' as metric_name, 'Par rapport à l’année passée' as display_name, 'fr' as language, 'Toutes les comparaisons d’une année sur l’autre sont effectuées par rapport à la même période sélectionnée il y a 12 mois. Ici, la période sélectionnée commence à partir de ' || {{date_start}} || ' à ' || {{date_end}} || ', Par rapport à l’année passée compare à la période de ' || toDate(addMonths({{date_start}}, -12)) || ' à ' ||  toDate(addMonths({{date_end}}, -12)) as value
```


    {% repeat
        id="glossary"
        data="metric_utilization"
        column="display_name"
        where="lower(language) = '{{$translations.inline_query}}'"
    %}
    ### {% value data="metric_utilization" value="display_name" filters=["glossary"] /%} 

    {% value data="metric_utilization" value="value" filters=["glossary"] /%}{% .text-sm %}
    {% /repeat %}
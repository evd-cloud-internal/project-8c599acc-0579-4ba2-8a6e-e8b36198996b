---
name: metric_employee_impact
assetId: 18613e41-627e-4939-b5e0-899197bdbf15
type: partial
---

---
metric_name_array: "('Average Time Saved', 'Time to Response to Therapy (Anxiety)', 'Time to Response To Therapy (Depression)', 'Medical Outcomes', 'Employee Well-Being Score', 'Employee Well-Being Focus Areas', 'Effectiveness of Care (Overall)', 'Average Well-Being Score Overtime')"
---
```sql metric_employee_impact
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
        data="metric_employee_impact"
        column="display_name"
        where="lower(language) = '{{$translations.inline_query}}'"
    %}
    ### {% value data="metric_employee_impact" value="display_name" filters=["glossary"]/%}

    {% value data="metric_employee_impact" value="value" filters=["glossary"]/%}{% .text-sm %}
    {% /repeat %}
---
name: OLD_total_sessions_program_big_value
assetId: 397cacc8-bf8c-4ecf-a9d0-8a87700d402a
type: partial
---

---
privacy_restriction_value: 5
---

```sql metric_desc_total_sessions
select
value as description
from base_client_reporting_metric_descriptions
where language = 'EN'
and metric_name = 'Total Sessions'
```
```sql restrictions_total_sessions_overall
select
current_sessions
from {{big_values_overview}} as big_values
```

{% if
    data="restrictions_total_sessions_overall"
    where="current_sessions >= {{$privacy_restriction_value}}"
%}

    {% big_value
        data="big_values_overview"
        value="current_sessions"
        comparison={
            compare_vs="target"
            target="prior_sessions"
            text="vs. last year"
            display_type="pct"
            delta=true
            pct_fmt="pct1"
        }
        title="Total Sessions"
        info="The number of daily interactions with the care team, including consultations and chats, excluding interaction with self-serve services, and automated follow-up chats when the members indicates ‘feeling better’."
    /%}

{% /if %}

{% else%}

    {% callout
    type="warning"
    %}
    Metric value(s) cannot be displayed because it is below privacy threshold
    {% /callout %}

{% /else%}


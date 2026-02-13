---
name: WIP_PJS
assetId: df2cc1dc-82e2-4d1a-806f-91d3385f878f
type: partial
---

{% partial
    file="combinefilters"
/%}

```sql pjs_q1
select
avg(response::float) as avg_score
, count(*) as volume
from fct_pjs_survey_responses_truncated
where question = 'How was your experience using our service?'
and date_day < date_trunc('month',today())
    and date_day <= {{date_end.selected}}
    and {{filter_value.filter}}
```

```sql pjs_q2
select
response
, count(*) as volume
, round(count(*) * 100.0 / sum(count(*)) over (), 1) as pct
from fct_pjs_survey_responses_truncated
where question = 'How much time away from work/school did the service we provided help you save?'
    and date_day < date_trunc('month',today())
    and date_day <= {{date_end.selected}}
    and {{filter_value.filter}}
group by 1
order by case response
    when 'no_time_saved' then 1
    when '1_to_3h' then 2
    when '3_a_5h' then 3
    when '5h_to_a_full_work_day' then 4
    when 'more_than_a_day' then 5
    else 6
end
```


```sql pjs_q3
select
avg(response::float) as avg_score
, count(*) as volume
from fct_pjs_survey_responses_truncated
where question = 'How helpful was the service we provided in improving your well-being?'
    and date_day < date_trunc('month',today())
    and date_day <= {{date_end.selected}}
    and {{filter_value.filter}}
```



### How was your exp. using our service?

{% row card=true %}


{% big_value
    data="pjs_q1"
    value="avg_score"
    title="Average Score"
/%}

{% big_value
    data="pjs_q1"
    value="volume"
    title="Number of responses"
/%}
{% /row %}


### How much time away from work/school did the service we provided help you save?

{% table
    data="pjs_q2"
%}
{% /table %}



### How helpful was the service we provided in improving your well-being?

{% row card=true %}

{% big_value
    data="pjs_q3"
    value="avg_score"
    title="Average Score"
/%}

{% big_value
    data="pjs_q3"
    value="volume"
    title="Number of responses"
/%}
{% /row %}
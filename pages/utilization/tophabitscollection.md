---
name: top_habits_collection
assetId: 7a44afa8-8dd3-488b-a71e-864d9941b15e
type: partial
---

```sql metric_calculation_top_habit_collections
select habit_collection_{{$translations.inline_query}} as habit_collection

      , count(case when date_month_est between {{date_start.selected}}
    and {{date_end.selected}} then habit_collection_{{$translations.inline_query}} end) as total_habits_marked_done_current

      , case when count(distinct case when date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then user_id end) >= {{$privacy_restriction_medium}}
        then count(case when date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then habit_collection_{{$translations.inline_query}} end)
        else null
        end as total_habits_marked_done_previous

      , count(distinct case when date_month_est between {{date_start.selected}}
    and {{date_end.selected}} then user_id end) as unique_users_current

      , case when 
        count(distinct case when date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then user_id end) >= {{$privacy_restriction_medium}}
        then count(distinct case when date_month_est between addMonths({{date_start.selected}}, -12) and addMonths({{date_end.selected}}, -12) then user_id end)
        else null
        end as unique_users_previous

from wellness_activities_with_contracts_truncated
where activity in ('habit_marked_done')
    and ((date_month_est between {{date_start.selected}}
    and {{date_end.selected}})
    or (date_month_est between addMonths({{date_start.selected}}, -12)
    and addMonths({{date_end.selected}}, -12)))
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
order by total_habits_marked_done_current desc
limit 5
```


```sql restriction_top_habit_collections
select habit_collection
    , total_habits_marked_done_current
    , total_habits_marked_done_previous
    , unique_users_current
    , unique_users_previous
from {{metric_calculation_top_habit_collections}}
where unique_users_current >= {{$privacy_restriction_medium}}
```


{% stack card=true %}

    ### {{$translations.section_descriptions.utilization.habits.title}}
    <!-- Habits -->
    <!-- Shows how members build and maintain healthy habits over time. -->
    {{$translations.section_descriptions.utilization.habits.overview}}

    {% if
        data="restriction_top_habit_collections"
        condition="has_rows"
        %}
        
        {% table
            data="restriction_top_habit_collections"
            order="total_habits_marked_done_current desc"
            title="{{$translations.metric_definitions.utilization.top_habit.title}}"
            info="{{$translations.metric_definitions.utilization.top_habit.description}}"
            info_link="#{{$translations.metric_definitions.utilization.top_habit.info_link}}"
            info_link_title="{{ $translations.read_more}}"
        %}

        {% dimension
            value="habit_collection"
            title="{{$translations.metric_definitions.utilization.top_habit.habit_collection}}"
        /%}
        {% dimension
            value="unique_users_current"
            title="{{$translations.metric_definitions.utilization.top_habit.unique_users}}"
            fmt="num"
        /%}
        {% measure
            value="unique_users_current"
            comparison={
                compare_vs="target"
                target="unique_users_previous"
                pct_fmt="pct1"
            }
            viz="delta"
            title=""
        /%}
        {% dimension
            value="total_habits_marked_done_current"
            title="{{$translations.metric_definitions.utilization.top_habit.habits_marked_done}}"
            info="{{$translations.metric_definitions.utilization.top_habit.habits_marked_done_description}}"
            fmt="num"
        /%}
        {% measure
            value="total_habits_marked_done_current"
            comparison={
                compare_vs="target"
                target="total_habits_marked_done_previous"
                pct_fmt="pct1"
            }
            viz="delta"
            title=""
        /%}


        {% /table %}

    {% /if %}

    {% else %}

        {% callout
        type="warning"
        %}
        {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.top_habit.title}}{{$translations.privacy_disclaimer_end}}
        {% /callout %}

    {% /else %}
    
{% /stack %}

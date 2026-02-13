---
name: registration_rate_comparison
assetId: e767084e-b1ab-44c5-a062-2047a565196c
type: partial
---

```sql metric_calculation_reg_rate_comp
with current_year as (
select
    date_month
    , sum(cast(max_registered_employees as decimal(10,3))) / sum(cast(max_eligible_members as decimal(10,2))) as registration_rate
    , sum(cast(max_registered_employees as decimal(10,3))) as reg_members
    ,  sum(cast(max_eligible_members as decimal(10,3))) as eligible_members
from client_reporting_monthly
where date_month < date_trunc('month',today())
and date_month >= {{date_start.selected}}
and date_month <= {{date_end.selected}}
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
order by 1 desc
limit 1
)

, last_year as (
select
    date_month
    , sum(cast(max_registered_employees as decimal(10,3))) / sum(cast(max_eligible_members as decimal(10,2))) as registration_rate_last_year
    , sum(cast(max_registered_employees as decimal(10,3))) as reg_members_last_year
    ,  sum(cast(max_eligible_members as decimal(10,3))) as eligible_members_last_year
from client_reporting_monthly
where date_month < date_trunc('month',today())
and date_month <= date_trunc('month',addMonths({{date_end.selected}}, -12))
and {{filter_value.filter}}
and reporting_block_id = '{{ $reporting_block_id }}'
group by 1
order by 1 desc
limit 1
)
select
registration_rate::float as registration_rate
, case
    when eligible_members_last_year < {{$privacy_restriction_high}} or reg_members_last_year < {{$privacy_restriction_low}} then null
    else registration_rate_last_year::float
end as registration_rate_last_year
, reg_members
, eligible_members
from current_year
left join last_year
    on date_trunc('month',addMonths(current_year.date_month, -12)) = last_year.date_month
```

```sql benchmark_extract_reg_rate_comp
with selected_account as (
select
distinct account_name
, case
    when {{comparison_filter.selected}} = 'Industry' then industry
    when {{comparison_filter.selected}} = 'Similar Organizations' then benchmark_group
    when {{comparison_filter.selected}} = 'Block of Business'
    then case when accounts.block_id = 'ccq' then 'dia'
        else accounts.block_id end
    else null
end as bench_group_join
, account_age_month
from {{filtered_table}} as filtered_table
left join accounts 
    on accounts.account_name = filtered_table.account_name
)

select
date_month
, benchmark_type
, case
    when benchmark_type = 'age_size' then 'Similar Organizations'
    when benchmark_type = 'industry' then 'Industry'
    when benchmark_type = 'pool' then 'Block of Business'
end as grouping_filter_map
, grouping
, account_age_month
, sample_size_member_funnel
, avg_registration_rate
from benchmark_types_monthly
inner join selected_account
    on  benchmark_types_monthly.grouping = selected_account.bench_group_join
where date_month < date_trunc('month', today())
and grouping_filter_map = {{comparison_filter.selected}}
and date_month = date_trunc('month',toDate({{date_end.selected}}))
```

```sql restrictions_reg_rate_comp_bench
select
reg_members
, eligible_members
, metric_calculation.registration_rate::float as reg_rate_metric
, case when account_age_month >= {{$privacy_restriction_lowest}} 
and sample_size_member_funnel >= {{$privacy_restriction_demographics_bucket}}
then bench_calculation.avg_registration_rate::float 
else null
end as reg_rate_bench
, account_age_month
, sample_size_member_funnel
from {{metric_calculation_reg_rate_comp}} as metric_calculation
cross join {{benchmark_extract_reg_rate_comp}} as bench_calculation
where eligible_members >= {{$privacy_restriction_high}} 
and reg_members >= {{$privacy_restriction_low}} 
```


```sql restrictions_reg_rate_comp
select
reg_members
, eligible_members
from {{metric_calculation_reg_rate_comp}} as metric_calculation
where eligible_members >= {{$privacy_restriction_high}} 
and reg_members >= {{$privacy_restriction_low}}
```

```sql is_one_account_or_no_bench_reg_rate_comp
select
{{comparison_filter.selected}} as bench_select_value
, count(distinct account_name) as account_count
from {{filtered_table}} as filtered_table
group by 1
```

```sql is_one_account_reg_rate_comp
select
count(distinct account_name) account_count
from {{filtered_table}} as filtered_table
```

```sql bench_or_time_reg_rate_comp
select
{{comparison_filter.selected}} as comparison_type

```

{% if
    data="is_one_account_or_no_bench_reg_rate_comp"
    where="(account_count > 1) or (bench_select_value = 'None')"
    %}
    <!-- multiple account selected or no bench selected -->

    {% if
        data="restrictions_reg_rate_comp"
        condition="has_rows"
        %}
        <!-- metric restrictions passed -->

        {% big_value
        data="metric_calculation_reg_rate_comp"
        value="registration_rate"
        title="{{$translations.metric_definitions.utilization.registration_rate.title}}"
        fmt="pct1"
        info="{{$translations.metric_definitions.utilization.registration_rate.description}}"
        info_link="#{{$translations.metric_definitions.utilization.registration_rate.info_link}}"
        info_link_title="{{ $translations.read_more}}"
        comparison={
            delta=true
            compare_vs="target"
            target="registration_rate_last_year"
            text="{{$translations.vs_last_year}}"
            pct_fmt="pct1"
        }
        /%}

    {% /if %}

    {% else %}
        <!-- metric restricrtions fail -->

        {% callout
        type="warning"
        %}
            {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.registration_rate.title}}{{$translations.privacy_disclaimer_end}}
        {% /callout %}

    {% /else %}

{% /if %}

{% else_if
    data="is_one_account_reg_rate_comp"
    where="account_count = 1"
    %}
    <!-- one account selected -->
    {% if
        data="bench_or_time_reg_rate_comp"
        condition="has_rows"
        where="comparison_type in ('Industry','Similar Organizations', 'Block of Business')"
        %}
        <!-- benchmark comparison selected -->

        {% if
            data="restrictions_reg_rate_comp_bench"
            condition="has_rows"
            %}
            <!-- metric AND bench pass restrictions -->

            {% big_value
                data="restrictions_reg_rate_comp_bench"
                value="reg_rate_metric"
                title="{{$translations.metric_definitions.utilization.registration_rate.title}}"
                fmt="pct1"
                comparison={
                    compare_vs="target"
                    target="reg_rate_bench"
                    text="{{$translations.vs_benchmark}}"
                    pct_fmt="pct1"
                }
                info="{{$translations.metric_definitions.utilization.registration_rate.description}}"
                info_link="#{{$translations.metric_definitions.utilization.registration_rate.info_link}}"
                info_link_title="{{ $translations.read_more}}"
            /%}

        {% /if %}

        {% else %}
            <!-- metric OR bench fail restrictions -->

            {% callout
            type="warning"
            %}
                {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.registration_rate.title}}{{$translations.privacy_disclaimer_end}}
            {% /callout %}
            
        {% /else %}

    {% /if %}

    {% else_if
        data="bench_or_time_reg_rate_comp"
        condition="no_rows"
        where="comparison_type in ('Industry','Similar Organizations')"
        %}
        <!-- time comparison selected -->

        {% if
            data="restrictions_reg_rate_comp"
            condition="has_rows"
            %}
            <!-- metric restrictions passed -->

            {% big_value
                data="metric_calculation_reg_rate_comp"
                value="registration_rate"
                fmt="pct1"
                title="{{$translations.metric_definitions.utilization.registration_rate.title}}"
                info="{{$translations.metric_definitions.utilization.registration_rate.description}}"
                info_link="#{{$translations.metric_definitions.utilization.registration_rate.info_link}}"
                info_link_title="{{ $translations.read_more}}"
                comparison={
                    compare_vs="target"
                    target="registration_rate_last_year"
                    delta=true
                    text="{{$translations.vs_last_year}}"
                    pct_fmt="pct1"
                }
            /%}

        {% /if %}

        {% else %}
            <!-- metric fail restrictions -->

            {% callout
            type="warning"
            %}
                {{$translations.privacy_disclaimer_start}} {{$translations.metric_definitions.utilization.registration_rate.title}}{{$translations.privacy_disclaimer_end}}
            {% /callout %}
            
        {% /else%}

    {% /else_if %}

{% /else_if %}



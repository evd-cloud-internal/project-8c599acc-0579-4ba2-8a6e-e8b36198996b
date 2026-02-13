---
name: OLD_ctd_utilization_rate
assetId: dfdecbdb-fdb5-494a-ab2d-e0781fa03ddf
type: partial
---

---
privacy_restriction_value_elig: 35
privacy_restriction_value_cases: 5
---

<!-- and date_month >= {{date_start.selected}}
    and date_month <= {{date_end.selected}} -->

```sql metric_calculation_ctd_util_rate
with aliases as (
select
    date_month
    , reporting_group_id
    , organization_id
    , sessions_primary_care as cases_primary_care
    , cases_mental_health
    , cases_eap
    , avg_eligible_members_primary_care
    , avg_eligible_members_mental_health
    , avg_eligible_members_eap
from client_reporting_monthly
where date_month < date_trunc('month',today())
    and {{filter_value.filter}}
    and reporting_block_id = '{{ $reporting_block_id }}'
)

, monthly as (
select
    date_month
    , sum(coalesce(aliases.cases_{{program_filter.literal}}, 0)) as cases
    , sum(coalesce(aliases.avg_eligible_members_{{program_filter.literal}}, 0)) as avg_eligible_members
    , min(accounts.number_of_months_since_renewal_{{program_filter.literal}}) as number_of_months_since_renewal
from aliases
left join organizations 
    on aliases.organization_id = organizations.organization_id
left join accounts
    on organizations.account_id = accounts.account_id
where aliases.date_month >= accounts.latest_renewal_date_{{program_filter.literal}}
group by 1
)

, total as ( 
select
    sum(cases) as cases
    , avg(avg_eligible_members) as avg_eligible_members
    , avg(number_of_months_since_renewal) as avg_number_of_months_since_renewal
from monthly
)

select
    round(toFloat64(cases)/ toFloat64(avg_eligible_members), 3) as ctd_util_rate
    , toFloat64(cases) as cases
    , toFloat64(avg_eligible_members) as avg_eligible_members
    , avg_number_of_months_since_renewal - 1 as renewal_months
from total
```

```sql restrictions_ctd_util_rate
select
cases
, avg_eligible_members
, renewal_months
, ctd_util_rate
from {{metric_calculation_ctd_util_rate}} as metric_calculation
where cases >= {{$privacy_restriction_value_cases}}
and avg_eligible_members >= {{$privacy_restriction_value_elig}}
and renewal_months >= 2
```



    <!-- one account selected -->
    {% if
        data="restrictions_ctd_util_rate"
        condition="has_rows"
        %}
        <!-- pass privacy restrictions -->

            {% big_value
            data="restrictions_ctd_util_rate"
            value="ctd_util_rate"
            title="Contract to Date"
            fmt="pct1"
            info="The cumulative utilization rate, based on completed months since contract launch or renewal."
            info_link="#contract-to-date-utilization-rate"
            info_link_title="Read more"
            /%}

    {% /if %}

    {% else%}
        <!-- fail privacy restrictions -->


        {% callout
        type="warning"
        %}
        Contract to Date Utilization Rate cannot be displayed because it is below privacy threshold
        {% /callout %}

    {% /else %}


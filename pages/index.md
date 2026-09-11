---
title: SF public compensation
---

<p class="text-sm text-gray-500 -mt-2 mb-6">
  Source: <a href="https://data.sfgov.org/City-Management-and-Ethics/Employee-Compensation/88g8-5mnd">DataSF – Employee Compensation</a>,
  data as of 2026-08-14.
</p>

```sql current_employee_comp
  select
    department,
    job,
    "employee name" as employee,
    try_cast(replace("total compensation", ',', '') as double) as total_comp,
    try_cast(replace(salaries, ',', '') as double)             as base_salary,
    try_cast(replace(overtime, ',', '') as double)             as overtime,
    try_cast(replace("other salaries", ',', '') as double)     as other_salaries,
    try_cast(replace("total salary", ',', '') as double)       as total_salary,
    try_cast(replace("total benefits", ',', '') as double)     as total_benefits,
    try_cast(replace(hours, ',', '') as double)                as hours
  from sf.employee_compensation
  where "year type" = 'Fiscal'
    and year = 2026
```

```sql total_comp_all_employees
select sum(total_comp)
from ${current_employee_comp}
```

```sql total_employees
select count(total_comp)::integer as employee_count
from ${current_employee_comp}
```

In 2026, San Francisco paid $<Value data={total_comp_all_employees} /> in total compensation to its <Value data={total_employees} /> employees.

## Top employee compensation

Many of these employees were very well compensated. Most of the top paid employees are in the Sheriff, Police, Public Health, Fire, and Retirement System Departments.

```sql top_compensation
select *
from ${current_employee_comp}
order by total_comp desc
limit 500
```

<DataTable data={top_compensation} search=true rows=50 />

## Departments

```sql histogram

with bucketed as (
  select
    department,
    floor(total_comp / 50000) * 50000 as bucket_start
  from ${current_employee_comp}
  where total_comp is not null
    and total_comp >= 0
),

counts as (
  select
    department,
    bucket_start,
    count(*) as employees
  from bucketed
  group by department, bucket_start
)

select
  department,
  bucket_start,
  employees,
  employees * 1.0 / sum(employees) over (partition by department) as pct_of_department
from counts
order by department, bucket_start
```

<BarChart
  data={histogram}
  x=bucket_start
  y=employees
  series=Department
  sort=false
/>

```department_summaries
select
  department,
  count(*)                                             as employees,
  sum(total_comp)                                      as total_comp_cost,
  avg(total_comp)                                      as avg_total_comp,
  median(total_comp)                                   as median_total_comp,
  quantile_cont(total_comp, 0.25)                      as p25_total_comp,
  quantile_cont(total_comp, 0.75)                      as p75_total_comp,
  max(total_comp)                                      as max_total_comp,
  avg(base_salary)                                     as avg_base_salary,
  avg(total_salary)                                    as avg_total_salary,
  avg(overtime)                                        as avg_overtime,
  sum(overtime) / nullif(sum(total_salary), 0)         as overtime_pct_of_pay,
  avg(total_benefits)                                  as avg_benefits,
  sum(total_benefits) / nullif(sum(total_comp), 0)     as benefits_pct_of_comp,
  avg(hours)                                           as avg_hours
from ${current_employee_comp}
where total_comp > 0
group by department
order by median_total_comp desc
```

<DataTable data={department_summaries} search=true rows=25 sort="median_total_comp desc">
  <Column id=Department />
  <Column id=employees fmt=num0 />
  <Column id=total_comp_cost title="Total Comp Cost" fmt=usd0 />
  <Column id=median_total_comp title="Median Comp" fmt=usd0 contentType=colorscale />
  <Column id=avg_total_comp title="Avg Comp" fmt=usd0 />
  <Column id=p25_total_comp title="25th Pct" fmt=usd0 />
  <Column id=p75_total_comp title="75th Pct" fmt=usd0 />
  <Column id=max_total_comp title="Max Comp" fmt=usd0 />
  <Column id=avg_base_salary title="Avg Base Salary" fmt=usd0 />
  <Column id=avg_overtime title="Avg Overtime" fmt=usd0 />
  <Column id=overtime_pct_of_pay title="OT % of Pay" fmt=pct1 />
  <Column id=benefits_pct_of_comp title="Benefits % of Comp" fmt=pct1 />
  <Column id=avg_hours title="Avg Hours" fmt=num0 />
</DataTable>

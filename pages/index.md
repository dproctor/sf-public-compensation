---
title: SF public compensation
---

<p class="text-sm text-gray-500 -mt-2 mb-6">
  Source: <a href="https://data.sfgov.org/City-Management-and-Ethics/Employee-Compensation/88g8-5mnd">DataSF – Employee Compensation</a>,
  data as of 2026-08-14.
</p>

```sql top_compensation
select
  Department,
  Job,
  "Employee Name",
  try_cast(replace("total compensation", ',', '') as double) as "Total Compensation",
  Salaries,
  Overtime,
  "Other Salaries",
  "Total Salary",
  Retirement,
  "Health and Dental",
  "Other Benefits",
  "Total Benefits",
from sf.employee_compensation
where "year type" = 'Fiscal'
    and year = (
      select max(year)
      from sf.employee_compensation
      where "year type" = 'Fiscal'
    )
order by "Total Compensation" desc
limit 500
```

<DataTable data={top_compensation} search=true rows=50 />

```sql histogram
with comp as (
  select
    department,
    try_cast(replace("total compensation", ',', '') as double) as total_comp
  from sf.employee_compensation
  where "year type" = 'Fiscal'
    and year = (
      select max(year)
      from sf.employee_compensation
      where "year type" = 'Fiscal'
    )
),

bucketed as (
  select
    department,
    floor(total_comp / 50000) * 50000 as bucket_start
  from comp
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
  '$' || cast(bucket_start / 1000 as integer) || 'k – $'
      || cast((bucket_start + 50000) / 1000 as integer) || 'k' as bucket_label,
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

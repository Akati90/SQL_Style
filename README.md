
# Akati's SQL Style Guide

Hi, everyone. I am teamlead of Data Analytic team in NurZholy Customs Service. This guide is NECESSARILY for all data analyst. Everyone in team has to write same in production mode to better understand other team members sql code and logic.

This guide is an attempt to document my preferences for formatting SQL in the hope that it may be of some use to others.  
 
## Example how its looks like, guide is below 

Here's a non-trivial query to give you an idea of what this style guide looks like in the practice:

```sql

with sr as ( -- sel_and_cus
select 
         v1.invoice_id
        ,v1.seller_bin
        ,count(1) cnt
from schema_esf.table_esf v1
where 1=1  
  and v1.status in ('status1', 'status2')
group by v1.invoice_id
        ,v1.seller_bin
)
,tr on (
select 
         v1.invoice_id
from schema_esf.table_esf v1
where 1=1  
  and v1.turnover_date >= date '2020-01-01'
group by v1.invoice_id 
)

select 
         v1.seller_bin
        ,v1.registration_number
        ,v1.invoice_id
        ,v1.related_invoice_id
        ,v1.related_reg_number
        ,sr.cnt as cnt_customers
        ,v1.turnover_date   as turnover_date
        ,cast(v1.invoice_id as Nullable(UInt64)) as invoice_id_UInt64
        
from schema_esf.table_esf v1
join tr      on tr.invoice_id = v1.invoice_id
left join sr on sr.invoice_id = v1.invoice_id 
            and sr.seller_bin = v1.seller_bin
where 1=1
  and v1.invoice_date >= date '2024-01-01'
  and v1.invoice_type in ('TYPE1') 
  and v1.invoice_id not in (select invoice_id from schema_esf.t_ya_esf_w_bad_status)

```
-------------------------------------------------------------------------------------------------------------------------------

## Guidelines

### Names of SQL objects

* All Tables created by NZCS employees(DA, DE, DBA, Devs ...) started by `t` and add initials of employee  `ya` (Yerkin Akbergenov)
* All Views by `v` and add initials of employee
* All Materialized Views by `v` and add initials of employee
* All Procedues by `p` and add initials of employee
* All Functions by `f` and add initials of employee
* All Indexes by `i` and add initials of employee
* All Trigers by `tg` and add initials of employee
* All Transactions by `tr` and add initials of employee
* All Package by `pkg` and add initials of employee

```sql
-- Good -- table from other source
select * from schema_name.users u

-- Good
select * from schema_name.t_ya_users_from_2020 u

-- Bad
select * from schema_name.users_from_2020 u

```

### Use lowercase SQL

It's just as readable as uppercase SQL and you won't have to constantly be holding down a shift key.

```sql
-- Good
select * from schema_name.users u

-- Bad
SELECT * FROM schema_name.users u

-- Bad
Select * From schema_name.users u
```

### Put each selected column on its own line

When selecting columns, always put each column name on its own line and never on the same line as `select`. For multiple columns, it's easier to read when each column is on its own line. And for single columns, it's easier to add additional columns without any reformatting (which you would have to do if the single column name was on the same line as the `select`). And use double tab for each column. 

```sql
-- Good
select 
        u.id
from schema_name.users u

-- Good
select 
        u.id,
        u.email,
        u.adress
from schema_name.users u

-- Bad -- one tab
select 
    u.id
from schema_name.users u


-- Bad -- same line as select
select  u.id
from schema_name.users u

-- Bad --
select u.id, u.email
from schema_name.users u

-- Bad -- each column each line
select 
        u.id, u.email
from schema_name.users u
```

## select *

When selecting `*` it's fine to include the `*` next to the `select` and also fine to include the `from` on the same line, assuming no additional complexity like `where` conditions:

```sql
-- Good
select * from schema_name.users 

-- Good too
select *
from schema_name.users

-- Bad
select * from schema_name.users where u.email = 'name@example.com'
```

## List of SQL main Operations and order them. 

We use in main pipelines and jobs only `select`, `from`, `join`, `left join`, `where`, `group by`, `limit`. And only in this order. First use all `joins` , then `left joins`. No `having` , `right join`, `outer join`, `cross join` etc.


```sql
-- Good 
select *
from schema_name.users u
join schema_name.orders o       on o.user_id = u.id
join schema_name.product p      on p.id      = o.product_id 
left join schema_name.regions r on r.user_id = o.region_id 

-- Good
select *
from schema_name.users u
join schema_name.orders o on o.user_id = u.id
where email = 'name@example.com' 

-- Bad
select *
from schema_name.users
join schema_name.orders o       on o.user_id = u.id
left join schema_name.regions r on r.user_id = o.region_id 
join schema_name.product p      on p.id      = o.product_id 

-- Bad
select *
from schema_name.users u
cross join schema_name.orders o on o.user_id = u.id
where u.email = 'name@example.com' 
```

### Use `join` with `on`. 

Try to write all join conditions at `on` segment. Forbidden join with `using` and join with comma in `from`

```sql
-- Good 
select *
from schema_name.users u
join schema_name.orders o       on o.user_id = u.id
join schema_name.product p      on p.id      = o.product_id  

-- Bad
select *
from schema_name.users
join schema_name.orders o       on using(user_id)

-- Bad
select *
from schema_name.users u, schema_name.orders o, schema_name.product p 
where o.user_id = u.id
  and p.id      = o.product_id  

```

## No space and tab before SQL main Operations. 

When selecting all main operations from beginning of line without space or tabs. For `and` use double space. For CTEs(subqueries in operator `with`) tabs are more preferable(not necessary).

```sql
-- Good too
select *
from schema_name.users

-- Good
select * 
from schema_name.users u
join schema_name.orders o on o.user_id = u.id
where u.email = 'name@example.com' 
  and u.date >= date '2020-01-01'

-- Good
with sr as ( -- sel_and_cus_CRP
    select 
             v1.invoice_id 
            ,count(1) as cnt
    from schema_esf.table_esf v1
    where 1=1  
      and v1.status in ('CREATED', 'DELIVERED','SEND_TO_ISGO','CANCELED_BY_OGD')
    group by v1.invoice_id 
)
select *
from sr

-- Bad
select * 
    from schema_name.users u
    where u.email = 'name@example.com'

-- Bad
select * 
    from schema_name.users u
        join schema_name.orders o on o.user_id = u.id
    where u.email = 'name@example.com'


-- Bad
select * 
    from schema_name.users u
        where u.email = 'name@example.com'

```

## Indenting where conditions

Similarly, conditions should always be spread across multiple lines to maximize readability and make them easier to add to. Operators `and`,`or` should be placed at the beginnig of each line:

```sql
-- Good
select *
from schema_name.users u
where 1=1
  and u.email = 'example@domain.com'

-- Good
select *
from schema_name.users u
where u.email like '%@domain.com' 
  and u.created_at >= '2021-10-08'

-- Bad
select *
from schema_name.users u
where u.email like '%@domain.com' and u.created_at >= '2021-10-08'

-- Bad
select *
from schema_name.users u
where u.email like '%@domain.com' and
      u.created_at >= '2021-10-08' 
```

### Use single quotes

Some SQL dialects like BigQuery support using double quotes, but for most dialects double quotes will wind up referring to column names. For that reason, single quotes are preferable:

```sql
-- Good
select *
from schema_name.users u
where u.email = 'example@domain.com'

-- Bad
select *
from schema_name.users u
where u.email = "example@domain.com"
```

If your SQL dialect supports double quoted strings and you prefer them, just make sure to be consistent and not switch between single and double quotes.

### Use `!=` over `<>`

Simply because `!=` reads like "not equal" which is closer to how we'd say it out loud.

```sql
-- Good
select 
        count(*) as paying_users_count
from schema_name.users u
where u.plan_name != 'free'

-- Very Bad
select 
        count(*) as paying_users_count
from schema_name.users u
where u.plan_name <> 'free'
```

### Commas could be equaly at the end of lines or at the beginning of lines. Try to use in code only one type comma style for entire pipeline. No space after comma

```sql
-- Good
select 
        u.id,
        u.email,
        u.adress,
from schema_name.users u

-- Good
select 
         u.id
        ,u.email
        ,u.adress
from schema_name.users u

-- Bad
select
        u.id
        , u.email
from schema_name.users u

-- Bad
select 
         u.id
        ,u.email,
         u.adress
from schema_name.users u
```

### Column name conventions

* Boolean fields should be prefixed with  `flag_`, `has_`,  or `does_`. For example, `is_customer`, `has_unsubscribed`, etc.
* Date-only fields should be suffixed with `_date`. For example, `report_date`.
* Date+time fields should be suffixed with `_dtime`. For example, `created_dtime`, `posted_dtime`, etc.

### Column order conventions

Put the primary key first, followed by foreign keys, then by all other columns. If the table has any system columns (`created_at`, `updated_at`, `is_deleted`, etc.), put those last.

```sql
-- Good
select
        u.id,
        u.name,
        u.created_at
from schema_name.users u

-- Bad
select
        u.created_at,
        u.name,
        u.id,
from schema_name.users u
```

When you have mutliple join conditions, place each one on their own indented line:

```sql
-- Good
select
        u.email,
        sum(c.amount) as total_revenue
from schema_name.users u
join schema_name.charges c on u.id = c.user_id 
                          and refunded = false
group by u.email
```

### Always aliasing table names.

Use table names like `users` to `u` and `charges` to `c`, dont write column without table short name even if it's only 1 table:

```sql
-- Good
select
        u.email,   
        sum(c.amount) as total_revenue
from schema_name.users u
inner join schema_name.charges c on u.id = c.user_id
group by u.email

-- Bad
select
        email,
        sum(amount) as total_revenue
from schema_name.users u
inner join schema_name.charges c on u.id = c.user_id
group by email

-- Bad
select
        email
from schema_name.users 
```


### Always rename aggregates and function-wrapped arguments

```sql
-- Good
select 
        count(*) as cnt_users
from schema_name.users

-- Good
select 
         count(*) as total_users
        ,sum(u.salary) as amt_users_salary
from schema_name.users u

-- Bad
select 
         count(*)
        ,sum(u.salary)
from schema_name.users u
```

### Be explicit in boolean conditions

```sql
-- Good
select * 
from schema_name.customers ct
where ct.is_cancelled = true

-- Good
select * 
from schema_name.customers ct
where ct.is_cancelled = false

-- Bad
select * 
from schema_name.customers ct
where ct.is_cancelled

-- Bad
select * 
from schema_name.customers ct
where not ct.is_cancelled
```

### Use `as` to alias column names

```sql
-- Good
select
        u.id,
        u.email,
        timestamp_trunc(u.created_at, 'month') as signup_month
from schema_name.users u

-- Bad
select
        u.id,
        u.email,
        timestamp_trunc(u.created_at, 'month') signup_month
from schema_name.users u
```

### Group using only column names, not numbers

If group using column, it has to be in select in same order:

```sql
-- Good
select 
        c.user_id, 
        count(*) as total_charges
from schema_name.charges c
group by c.user_id

-- Bad -- 1
select 
        c.user_id, 
        count(*) as total_charges
from schema_name.charges c
group by 1

-- Bad -- no user_id in select
select 
        count(*) as total_charges
from schema_name.charges c
group by c.user_id

-- Bad -- order of columns is different
select  
        c.user_id,
        c.region,
        count(*) as total_charges
from schema_name.charges c
group by c.region, 
         c.user_id
```

### Grouping columns should go first

```sql
-- Good
select 
        c.user_id, 
        count(*) as total_charges
from schema_name.charges c
group by c.user_id

-- Good
select  
        c.region, 
        c.user_id,
        count(*) as total_charges
from schema_name.charges c
group by c.region, 
         c.user_id

-- Bad 
select 
         count(*) as total_charges
        ,c.user_id 
from schema_name.charges c
group by c.user_id

-- Bad
select  
        c.region, 
        count(*) as total_charges,
        c.user_id
from schema_name.charges c
group by c.region, 
         c.user_id
```

### Aligning case/when statements

Each `when` should be on its own line (first `when` starts in the `case` line). The `then` can be on the same line or on its own line below it, just aim to be consistent. Dont use `if` statement.

```sql
-- Good
select
        case when e.event_name = 'viewed_homepage' then 'Homepage'
             when e.event_name = 'viewed_editor'   then 'Editor'
             else 'Other'
        end as page_name
from schema_name.events e

-- Good too
select
        case when e.event_name = 'viewed_homepage' 
             then 'Homepage'
             when e.event_name = 'viewed_editor'   
             then 'Editor'
             else 'Other'
             end as page_name
from schema_name.events e too

-- Bad 
select
        case 
            when e.event_name = 'viewed_homepage' then 'Homepage'
            when e.event_name = 'viewed_editor'   then 'Editor'
                else 'Other'
        end as page_name
from schema_name.events e
```

### Double case and with statements

Dont use second case in case statement. Same, dont use `with` in `with`. Its SQL, not JAVA.

```sql
-- Good
select
        case when e.event_name = 'viewed_homepage' and e.code  = 1 then 'Homepage'
             when e.event_name = 'viewed_homepage' and e.code != 1 then 'Homepage_old'
             when e.event_name = 'viewed_editor'   and e.code  = 2 then 'Editor'
             else 'Other'
        end as page_name
from schema_name.events e

-- Good  
with s1 as (
    select ...
), s2 as (
    select ...
    from s1
)
select *
from s2

-- Bad
select
        case when e.event_name = 'viewed_homepage' 
             then case when e.code  = 1 then 'Homepage' 
                       when e.code != 1 then 'Homepage_old'
                       end 
             when e.event_name = 'viewed_editor'
             then case when e.code  = 2 then 'Editor'  
                       end 
             else 'Other'
        end as page_name
from schema_name.events e


-- Bad 
with s2 as (
    with s1 as (
        select ...
        )
    select *
    from s1
)
select *
from s2
```

### Window functions

Leave it all on its own line:

```sql
-- Good
select
        b.user_id,
        b.name,
        row_number()over(partition by b.user_id order by b.date_updated desc) as details_rank
from schema_name.billing_stored_details b

-- Okay, but worse 
select
        b.user_id,
        b.name,
        row_number() over (
            partition by b.user_id
            order by b.date_updated desc
        ) as details_rank
from schema_name.billing_stored_details b
```

### If its possible Try to use CTEs, not subqueries. But in some cases subqueries are faster, then should use subqueries.

Avoid subqueries; CTEs will make your queries easier to read and reason about.
When using CTEs, pad the query with new lines. 
Closing CTE parentheses should use the same indentation level as `with` and the CTE names.

```sql
-- Good
with ordered_details as (
    select
            b.user_id,
            b.name,
            row_number() over (partition by b.user_id order by b.date_updated desc) as details_rank
    from schema_name.billing_stored_details b
),
first_updates as (
    select 
            od.user_id, 
            od.name
    from schema_name.ordered_details od
    where od.details_rank = 1
)
select * 
from first_updates

-- Bad
select 
        user_id, 
        name
from (
    select
            b.user_id,
            b.name,
            row_number() over (partition by b.user_id order by b.date_updated desc) as details_rank
    from schema_name.billing_stored_details b
) ranked
where details_rank = 1
```


### Distinct

Use `group by` instead of `distinct`, `distinct` is ok only in `count` statement:

```sql
-- Good -- unique users
select 
        c.user_id, 
from schema_name.charges c
group by c.user_id

-- Good
select  
         c.region
        ,count(distinct c.user_id) as cnt_dist_users
from schema_name.charges c
group by c.region

-- Bad 
select 
        distinct c.user_id 
from schema_name.charges c
```

### Schema name

Write full table name with schema name:

```sql
-- Good  
select 
        c.user_id 
from schema_name.charges c

-- Bad 
select 
        c.user_id 
from charges c
```

### Truncate instead of delete table

Always use  `truncate table`:

```sql
-- Good  
truncate table schema_name.charges;

-- Bad 
delete table schema_name.charges;
```

### Between is not good

Just use `>= and <=`:

```sql
-- Good
select
        u.id,
        u.email
from schema_name.users u
where 1=1
  and u.created_at >= date '2020-01-01'
  and u.created_at <  date '2021-01-01'

-- Bad 
select
        u.id,
        u.email
from schema_name.users u
where 1=1
  and u.created_at between date '2020-01-01' and date '2021-01-01'
```

### Double partition 

Use `()over(partition by)` one at a time:

```sql
-- Good  
select
            b.user_id,
            b.name,
            row_number()over(partition by b.user_id order by b.date_updated desc) as details_rank
from schema_name.billing_stored_details b

-- Bad 
select
            b.user_id,
            rank()over(partition by ...) - count(1)over(partition ...)
from schema_name.billing_stored_details b

```

### Dont use subqueries in Select statement

```sql
-- Good  
select
         u.id
        ,u.email
        ,avg(salary)over(partition by u.user_id) as avg_sal
from schema_name.users u

-- Bad 
select
         u.id
        ,u.email
        ,(select avg(salary) from  schema_name.users ) as avg_sal
from schema_name.users u
```

### Avoid using SQL syntax and system names in naming schema/table/column.

```sql
-- Good 
select
         u.id
        ,u.birth_date as date_
        ,date_trunc('month', u.birth_date) as month_
        ,date_trunc('month', u.birth_date) as mm
from schema_name.users u

-- Good 
select
         count(1) as cnt
        ,sum(salary) as amt
        ,avg(salary) as avg_
from schema_name.users u

-- Good 
create table schema_name.function_for_user as ...

-- Bad 
select
         u.id
        ,u.birth_date as date
        ,date_trunc('month', u.birth_date) as month
from schema_name.users u

-- Bad 
select
         count(1) as count
        ,sum(salary) as sum
        ,avg(salary) as avg
from schema_name.users u

-- Bad 
create table schema_name.function as ...
```


### date format `date 'yyyy-mm-dd'` 

It's just as readable. Only exception if SQL dialect cannot read this format. 

```sql
-- Good
select * 
from schema_name.users u
where u.date_updated >= date '2020-01-01'

-- Bad 
select * 
from schema_name.users u
where u.date_updated >=  '2020-01-01'

-- Bad 
select * 
from schema_name.users u
where u.date_updated >= to_date('2020/01/01', 'yyyy/mm/dd') 

-- Bad 
select * 
from schema_name.users u
where u.date_updated >= toDateTime('2020-01-01 23:00:00') 
```

### Union all

Union all generates less errors and misscalculation. Never mix different unions in one select statement. 

```sql
-- Good
select 
         u.name
        ,u.email
from schema_name.users u
union all 
select 
         u.name
        ,u.email
from schema_name.old_users u
union all 
select 
         u.name
        ,u.email
from schema_name.deleted_users u


-- Bad 
select 
         u.name
        ,u.email
from schema_name.users u
union all 
select 
         u.name
        ,u.email
from schema_name.old_users u
union distinct  
select 
         u.name
        ,u.email
from schema_name.deleted_users u
```




# Akati's SQL Style Guide

Hi, everyone. I am teamlead of Data Analytic team in NurZhol Custom Service. This guide is NECESSARILY for all data analyst. Everyone in team has to write same in production mode to better understand other team members sql code and logic.

This guide is an attempt to document my preferences for formatting SQL in the hope that it may be of some use to others.  
 
## Example

Here's a non-trivial query to give you an idea of what this style guide looks like in the practice:

```sql

with sr as ( -- sel_and_cus_CRP
    select 
             v1.invoice_id
            ,v1.seller_tin
            ,v1.invoice_type
            ,count(1) as cnt
    from schema_esf.table_esf v1
    where 1=1  
      and v1.status in ('CREATED', 'DELIVERED','SEND_TO_ISGO','CANCELED_BY_OGD')
    group by v1.invoice_id
            ,v1.seller_tin
            ,v1.invoice_type
)
select 
         v1.seller_tin
        ,v1.registration_number
        ,v1.invoice_id
        ,v1.related_invoice_id
        ,v1.related_reg_number
        ,v1.turnover_date   as turnover_date
        ,cast(v1.invoice_id as Nullable(UInt64)) as invoice_id_UInt64
        
from schema_esf.table_esf v1
left join sr on sr.invoice_id = v1.invoice_id 
            and sr.seller_tin = v1.seller_tin
where 1=1
  and v1.turnover_date >= date '2024-01-01'
  and v1.invoice_type in ('ORDINARY_INVOICE') 
  and v1.invoice_id not in (select invoice_id from schema_esf.t_ya_esf_w_bad_status)

```
## Guidelines

### Use lowercase SQL

It's just as readable as uppercase SQL and you won't have to constantly be holding down a shift key.

```sql
-- Good
select * from users u

-- Bad
SELECT * FROM users u

-- Bad
Select * From users u
```

### Put each selected column on its own line

When selecting columns, always put each column name on its own line and never on the same line as `select`. For multiple columns, it's easier to read when each column is on its own line. And for single columns, it's easier to add additional columns without any reformatting (which you would have to do if the single column name was on the same line as the `select`). And use double tab for each column. 

```sql
-- Good
select 
        u.id
from users u

-- Good
select 
        u.id,
        u.email,
        u.adress
from users u

-- Bad -- one tab
select 
    u.id
from users u


-- Bad -- same line as select
select  u.id
from users u

-- Bad --
select u.id, u.email
from users u

-- Bad -- each column each line
select 
        u.id, u.email
from users u
```

## select *

When selecting `*` it's fine to include the `*` next to the `select` and also fine to include the `from` on the same line, assuming no additional complexity like `where` conditions:

```sql
-- Good
select * from users 

-- Good too
select *
from users

-- Bad
select * from users where u.email = 'name@example.com'
```

## List of SQL main Operations and order them. 

We use in main pipelines and jobs only `select`, `from`, `join`, `left join`, `where`, `group by`, `limit`. And only in this order. First use all `joins` , then `left joins`. No `having` , `right join`, `outer join`, `cross join` etc.


```sql
-- Good 
select *
from users
join orders o       on o.user_id = u.id
join product p      on p.id      = o.product_id 
left join regions r on r.user_id = o.region_id 

-- Good
select *
from users u
join orders o on o.user_id = u.id
where email = 'name@example.com' 

-- Bad
select *
from users
join orders o       on o.user_id = u.id
left join regions r on r.user_id = o.region_id 
join product p      on p.id      = o.product_id 

-- Bad
select *
from users u
cross join orders o on o.user_id = u.id
where u.email = 'name@example.com' 
```

When selecting all main operations from beginning of line without space or tabs. For `and` use double space. For CTEs(subqueries in operator `with`) tabs are more preferable(not necessary).

```sql
-- Good too
select *
from users

-- Good
select * 
from users u
join orders o on o.user_id = u.id
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
    from users u
    where u.email = 'name@example.com'

-- Bad
select * 
    from users u
        join orders o on o.user_id = u.id
    where u.email = 'name@example.com'


-- Bad
select * 
    from users u
        where u.email = 'name@example.com'

```

## Indenting where conditions

Similarly, conditions should always be spread across multiple lines to maximize readability and make them easier to add to. Operators `and`,`or` should be placed at the beginnig of each line:

```sql
-- Good
select *
from users u
where 1=1
  and u.email = 'example@domain.com'

-- Good
select *
from users u
where u.email like '%@domain.com' 
  and u.created_at >= '2021-10-08'

-- Bad
select *
from users u
where u.email like '%@domain.com' and u.created_at >= '2021-10-08'

-- Bad
select *
from users u
where u.email like '%@domain.com' and
      u.created_at >= '2021-10-08' 
```

### Use single quotes

Some SQL dialects like BigQuery support using double quotes, but for most dialects double quotes will wind up referring to column names. For that reason, single quotes are preferable:

```sql
-- Good
select *
from users u
where u.email = 'example@domain.com'

-- Bad
select *
from users u
where u.email = "example@domain.com"
```

If your SQL dialect supports double quoted strings and you prefer them, just make sure to be consistent and not switch between single and double quotes.

### Use `!=` over `<>`

Simply because `!=` reads like "not equal" which is closer to how we'd say it out loud.

```sql
-- Good
select 
        count(*) as paying_users_count
from users u
where u.plan_name != 'free'

-- Very Bad
select 
        count(*) as paying_users_count
from users u
where u.plan_name <> 'free'
```

### Commas could be equaly at the end of lines or at the beginning of lines. Try to use in code only one type comma style for entire pipeline. No space after comma

```sql
-- Good
select 
        u.id,
        u.email,
        u.adress,
from users u

-- Good
select 
         u.id
        ,u.email
        ,u.adress
from users u

-- Bad
select
        u.id
        , u.email
from users u

-- Bad
select 
         u.id
        ,u.email,
         u.adress
from users u
```

### Column name conventions

* Boolean fields should be prefixed with `is_`, `has_`, or `does_`. For example, `is_customer`, `has_unsubscribed`, etc.
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
from users u

-- Bad
select
        u.created_at,
        u.name,
        u.id,
from users u
```

When you have mutliple join conditions, place each one on their own indented line:

```sql
-- Good
select
        u.email,
        sum(c.amount) as total_revenue
from users u
join charges c on u.id = c.user_id 
              and refunded = false
group by u.email
```

### Always aliasing table names.

Use table names like `users` to `u` and `charges` to `c`, dont write column without table short name even if it's only 1 table:

```sql
-- Good
select
        u.email,
        sum(charges.amount) as total_revenue
from users
inner join charges on u.id = c.user_id
group by u.email

-- Bad
select
        email,
        sum(amount) as total_revenue
from users u
inner join charges c on u.id = c.user_id
group by email

-- Bad
select
        email
from users 
```


### Always rename aggregates and function-wrapped arguments

```sql
-- Good
select 
        count(*) as cnt_users
from users

-- Good
select 
         count(*) as total_users
        ,sum(u.salary) as amt_users_salary
from users u

-- Bad
select 
         count(*)
        ,sum(u.salary)
from users u
```

### Be explicit in boolean conditions

```sql
-- Good
select * 
from customers ct
where ct.is_cancelled = true

-- Good
select * 
from customers 
where ct.is_cancelled = false

-- Bad
select * 
from customers ct
where ct.is_cancelled

-- Bad
select * 
from customers ct
where not ct.is_cancelled
```

### Use `as` to alias column names

```sql
-- Good
select
        u.id,
        u.email,
        timestamp_trunc(u.created_at, 'month') as signup_month
from users u

-- Bad
select
        u.id,
        u.email,
        timestamp_trunc(u.created_at, 'month') signup_month
from users u
```

### Group using only column names, not numbers

If group using column, it has to be in select in same order

```sql
-- Good
select 
        c.user_id, 
        count(*) as total_charges
from charges c
group by c.user_id

-- Bad -- 1
select 
        c.user_id, 
        count(*) as total_charges
from charges c
group by 1

-- Bad -- no user_id in select
select 
        count(*) as total_charges
from charges c
group by c.user_id

-- Bad -- order of columns is different
select  
        c.user_id,
        c.region
        count(*) as total_charges
from charges c
group by c.region, 
         c.user_id
```

### Grouping columns should go first

```sql
-- Good
select 
        c.user_id, 
        count(*) as total_charges
from charges c
group by c.user_id

-- Good
select  
        c.region, 
        c.user_id,
        count(*) as total_charges
from charges c
group by c.region, 
         c.user_id

-- Bad 
select 
         count(*) as total_charges
        ,c.user_id, 
from charges c
group by c.user_id

-- Bad
select  
        c.region, 
        count(*) as total_charges,
        c.user_id,
from charges c
group by c.region, 
         c.user_id
```

### Aligning case/when statements

Each `when` should be on its own line (nothing on the `case` line). The `then` can be on the same line or on its own line below it, just aim to be consistent. Dont use `if` statement.

```sql
-- Good
select
        case when e.event_name = 'viewed_homepage' then 'Homepage'
             when e.event_name = 'viewed_editor'   then 'Editor'
             else 'Other'
        end as page_name
from events e

-- Good too
select
        case when e.event_name = 'viewed_homepage' 
             then 'Homepage'
             when e.event_name = 'viewed_editor'   
             then 'Editor'
             else 'Other'
             end as page_name
from events e too

-- Bad 
select
        case 
            when e.event_name = 'viewed_homepage' then 'Homepage'
            when e.event_name = 'viewed_editor'   then 'Editor'
                else 'Other'
        end as page_name
from events e
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
from events e

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
from events e


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
from billing_stored_details b

-- Okay, but worse 
select
        b.user_id,
        b.name,
        row_number() over (
            partition by b.user_id
            order by b.date_updated desc
        ) as details_rank
from billing_stored_details b
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
    billing_stored_details b
),
first_updates as (
    select 
            od.user_id, 
            od.name
    from ordered_details od
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
    from billing_stored_details b
) ranked
where details_rank = 1
```

# Akati's SQL Style Guide

Hi, everyone. I am teamlead of Data Analytic team in NurZhol Custom Service. This guide is NECESSARILY for all data analyst. Everyone in team has to write same in production mode to better understand other team members sql code and logic.

This guide is an attempt to document my preferences for formatting SQL in the hope that it may be of some use to others.  
 
## Example

Here's a non-trivial query to give you an idea of what this style guide looks like in the practice:

```sql

with sr as ( -- sel_and_cus_CRP
    select 
             invoice_id
            ,seller_tin
            ,invoice_type
            ,count(1) as cnt
    from schema_esf.table_esf v1
    where 1=1  
      and v1.status in ('CREATED', 'DELIVERED','SEND_TO_ISGO','CANCELED_BY_OGD')
    group by invoice_id
            ,seller_tin
            ,invoice_type
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
select * from users

-- Bad
SELECT * FROM users

-- Bad
Select * From users
```

### Put each selected column on its own line

When selecting columns, always put each column name on its own line and never on the same line as `select`. For multiple columns, it's easier to read when each column is on its own line. And for single columns, it's easier to add additional columns without any reformatting (which you would have to do if the single column name was on the same line as the `select`). And use double tab for each column. Comma is equaly OK for before and after column

```sql
-- Good
select 
        id
from users 

-- Good
select 
        id,
        email,
        adress
from users 

-- Good
select 
         id
        ,email
        ,adress
from users

-- Bad -- one tab
select 
    id
from users 


-- Bad -- same line as select
select  id
from users 

-- Bad --
select id, email
from users

-- Bad -- each column each line
select 
        id, email
from users 
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
select * from users where email = 'name@example.com'
```

## List of SQL main Operations and order them. 

We use in main pipelines and jobs only select, from, join, left join, where, group by, limit. And only in this order. First use all joins , then left joins. No having , right join, outer join, cross join etc.


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
where email = 'name@example.com' 
```


When selecting all main opearations from beginning without space or tabs. For 'and' use double space.

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

-- Bad
select * 
    from users 
    where email = 'name@example.com'

-- Bad
select * 
    from users 
        join orders o on o.user_id = u.id
    where email = 'name@example.com'


-- Bad
select * 
    from users 
        where email = 'name@example.com'

```
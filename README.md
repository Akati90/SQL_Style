
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


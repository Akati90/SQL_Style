
# Akati's SQL Style Guide

Hi, everyone. I am teamlead of Data Analytic team in NurZhol Custom Service. This guide is NECESSARILY for all data analyst. Everyone in team has to write same in production mode to better understand other team members sql code and logic.

This guide is an attempt to document my preferences for formatting SQL in the hope that it may be of some use to others.  
 
## Example

Here's a non-trivial query to give you an idea of what this style guide looks like in the practice:

```sql
with hubspot_interest as (

    select
        email,
        timestamp_millis(property_beacon_interest) as expressed_interest_at
    from hubspot.contact
    where 
        property_beacon_interest is not null

), 

select * from first_interest
```
## Guidelines



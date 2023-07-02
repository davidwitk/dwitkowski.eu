---
title: "Better SQL is possible - DuckDB's SQL Flavor"
date: 2023-07-02
draft: false
showTableOfContents: false
---

The hype around DuckDB is growing, lots of people are talking and writing about it. Realizing that this might be a really big thing I started to play around recently with DuckDB myself.
I won't explain what DuckDB is here because others have already done this better than I could (e.g. [here](https://mattpalmer.io/posts/whats-the-hype-duckdb/) or [here](https://dagster.io/blog/duckdb-data-lake)).
Rather, I will share what I - as somebody writing lots of SQL every day - find super exciting: DuckDB is a completely new and modern DB engine, allowing the developers to introduce very cool 
SQL features that I truly miss on other platforms.

You can follow along the SQL code in [this notebook](https://colab.research.google.com/drive/15c27T207JfFmI8QjdrXUIbSF046Y6E5-?usp=sharing).

## Read and import files

Reading files to and from databases is surprisingly difficult. With DuckDB, this is super easy: 

```sql
select * from read_csv_auto('https://raw.githubusercontent.com/dbt-labs/jaffle_shop/main/seeds/raw_customers.csv') limit 5;
```

Not using CSVs, but Parquet files? Sure, no problem: 

```sql
select * from 'test.parquet';
```

This is the basic ingredient for using DuckDB as a lakehouse and for the [Modern-Data-Stack-in-a-Box approach](https://duckdb.org/2022/10/12/modern-data-stack-in-a-box.html): 
- Store data in S3 buckets
- Read and write directly with DuckDB

## Omit `select *`

Ever wondered why you always have to write `select *` if you want to take a quick look at a table, kind of boilerplate code? With DuckDB, you dont't have to do that! 

```sql
from payments;
```

## Exclude columns in `select`

Sometimes, you do need all columns except some. In this case, you have to enumerate all the columns which might end up in a pretty long list. 
With DuckDB, you can do the reverse and exclude columns you do not need:

```sql
select * exclude(id) from payments limit 5;
```

## Replace columns in `select`

Also, in some cases, if you want to select a column and change it on-the-fly by keeping the same alias, you can do so with `replace`:

```sql
select * replace(upper(payment_method) as payment_method, amount / 10 as amount) from payments limit 5;
```

## Group by all columns

I am fan of using numbers instead of column aliases in the group by statement because it allows for more flexibility. Still, you have to be exact about the number of columns to group by. 
In DuckDB's SQL flavor, there exists the handy `group by all`:   

```sql
select
    date_trunc('month', order_date :: date) as date_month,
    status,
    count(*) as count
from orders
group by all
```

## `filter` clause

Filtered measures are not a first-class citizen in most SQL dialects. You have to use case-when-statements if you want to filter within a measure and not the whole data set. 
DuckDB offers an explicit equivalent for this operation: 

```sql
select
    date_trunc('month', order_date :: date) as date_month,
    count(*) as count,
    count(*) filter (where status = 'completed') as count_completed,
    sum(case when status = 'completed' then 1 else 0 end) :: int as count_completed_standard -- case-when equivalent
from orders
group by all
```

## `qualify`: Filtering by a window functions within the same CTE

Window functions are cool and powerful. However, if you try to filter in a where statement by a window function within the same CTE, you get an SQL error. 
The classic solution is to set the filter in a following CTE. With DuckDB, you can set the filter within the same CTE with the `qualify` statement (Snowflake has this feature, too): 

```sql
select
    status,
    order_date,
    row_number() over (partition by status order by order_date :: date) = 1 as is_first
from orders
qualify is_first
```

## `asof` Joins

This one is super cool. Say you want to join together two fact tables or event streams, and you need the latest entry of one table that occurred before a record in the other table. 
The classic approach is to use an unequal `left join`, resulting in a fanout, and then to use window function to flag the desired final record which can then be filtered in
the following CTE. With DuckDB's `asof` Joins, all this is possible with the addition of a simple keyword!

Here is an example query with some sample data:
```sql
with

renewals as (

select '6001edda120cbf91c959920c' as contract_id, '2021-01-15' :: date as renewal_date
union all
select '6001edda120cbf91c959920c' as contract_id, '2022-01-15' :: date as renewal_date
union all
select '6001edda120cbf91c959920c' as contract_id, '2023-01-15' :: date as renewal_date

),

subs as (

select '2021-05-01' :: date as date_day, '6001edda120cbf91c959920c' as contract_id
union all
select '2021-07-01' :: date, '6001edda120cbf91c959920c' as contract_id
union all
select '2022-01-01' :: date, '6001edda120cbf91c959920c' as contract_id
union all
select '2022-01-16' :: date, '6001edda120cbf91c959920c' as contract_id
union all
select '2023-01-14' :: date, '6001edda120cbf91c959920c' as contract_id
union all
select '2024-01-01' :: date, '6001edda120cbf91c959920c' as contract_id
)

select
    subs.contract_id,
    subs.date_day,
    renewals.renewal_date
from subs
asof left join renewals on subs.date_day >= renewals.renewal_date
order by 2, 3
```

## `union by name`

Who has not tried to union two data sets, but struggling to bring the columns in both sets in the same order? With DuckDB's `union by name`, the name instead of the order 
of columns is relevant for the operation: 

```sql
select * from payments
union by name
select * from orders;

```

## Sample data

Take a quick look at the data? Usually, a mix of `limit`, `order by` and `filter`. DuckDB, let's you sample data randomly - pretty convenient! 

```sql
select * from orders using sample 10;
```


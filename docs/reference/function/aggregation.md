---
title: Aggregate functions
sidebar_label: Aggregate
description: Aggregate functions reference documentation.
---

This page describes the available functions to assist with performing aggregate
calculations.

## avg

`avg(value)` calculates simple average of values ignoring missing data (e.g
`null` values).

**Arguments:**

- `value` is any numeric value.

**Return value:**

Return value type is `double`.

**Examples:**

```questdb-sql title="Average transaction amount"
SELECT avg(amount) FROM transactions;
```

| avg  |
| :--- |
| 22.4 |

```questdb-sql title="Average transaction amount by payment_type"
SELECT payment_type, avg(amount) FROM transactions;
```

| cash_or_card | avg   |
| :----------- | :---- |
| cash         | 22.1  |
| card         | 27.4  |
| null         | 18.02 |

## count

`count()` or `count(*)` - counts rows irrespective of underlying data.

**Arguments:**

- `count` does not require arguments.

**Return value:**

Return value type is `long`.

**Examples:**

- Count of rows in the transactions table.

```questdb-sql
SELECT count() FROM transactions;
```

| count |
| :---- |
| 100   |

- Count of rows in the transactions table aggregated by `payment_type` value.

```questdb-sql
SELECT payment_type, count() FROM transactions;
```

| cash_or_card | count |
| :----------- | :---- |
| cash         | 25    |
| card         | 70    |
| null         | 5     |

:::note

`null` values are aggregated with `count()`.

:::

## count_distinct

`count_distinct(column_name)` - counts distinct values in `string`, `symbol`,
`long256`, `long`, or `int` columns.

**Return value:**

Return value type is `long`.

**Examples:**

- Count of distinct sides in the transactions table. Side column can either be
  `BUY` or `SELL` or `null`

```questdb-sql
SELECT count_distinct(side) FROM transactions;
```

| count_distinct |
| :------------- |
| 2              |

- Count of distinct counterparties in the transactions table aggregated by
  `payment_type` value.

```questdb-sql
SELECT payment_type, count_distinct(counterparty) FROM transactions;
```

| cash_or_card | count_distinct |
| :----------- | :------------- |
| cash         | 3              |
| card         | 23             |
| null         | 5              |

:::note

`null` values are not counted in the `count_distinct` function.

:::

## first/last

- `first(column_name)` - returns the first value of a column.
- `last(column_name)` - returns the last value of a column.

Supported column datatype: `double`, `float`, `integer`, `character`, `short`,
`byte`, `timestamp`, `date`, `long`, `geohash`.

If a table has a [designated timestamp](/docs/concept/designated-timestamp),
then the first row is always the row with the lowest timestamp (oldest) and the
last row is always the one with the highest (latest) timestamp. For a table
without a designated timestamp column, `first` returns the first row and `last`
returns the last inserted row, regardless of any timestamp column.

**Return value:**

Return value type is `string`.

**Examples:**

Given a table `sensors`, which has a designated timestamp column:

| device_id  | temperature | ts                          |
| :--------- | :---------- | :-------------------------- |
| arduino-01 | 12          | 2021-06-02T14:33:19.970258Z |
| arduino-02 | 10          | 2021-06-02T14:33:21.703934Z |
| arduino-03 | 18          | 2021-06-02T14:33:23.707013Z |

The following query returns oldest value for the `device_id` column:

```questdb-sql
SELECT first(device_id) FROM sensors;
```

| first      |
| :--------- |
| arduino-01 |

The following query returns the latest symbol value for the `device_id` column:

```questdb-sql
SELECT last(device_id) FROM sensors;
```

| last       |
| :--------- |
| arduino-03 |

Without selecting a designated timestamp column, the table may be unordered and
the query may return different result. Given an unordered table
`sensors_unordered`:

| device_id  | temperature | ts                          |
| :--------- | :---------- | :-------------------------- |
| arduino-01 | 12          | 2021-06-02T14:33:19.970258Z |
| arduino-03 | 18          | 2021-06-02T14:33:23.707013Z |
| arduino-02 | 10          | 2021-06-02T14:33:21.703934Z |

The following query returns the first record for the `device_id` column:

```questdb-sql
SELECT first(device_id) FROM sensors_unordered;
```

| first      |
| :--------- |
| arduino-01 |

The following query returns the last record for the `device_id` column:

```questdb-sql
SELECT last(device_id) FROM sensors_unordered;
```

| last       |
| :--------- |
| arduino-02 |

## haversine_dist_deg

`haversine_dist_deg(lat, lon, ts)` - calculates the traveled distance for a
series of latitude and longitude points.

**Arguments:**

- `lat` is the latitude expressed as degrees in decimal format (`double`)
- `lon` is the longitude expressed as degrees in decimal format (`double`)
- `ts` is the `timestamp` for the data point

**Return value:**

Return value type is `double`.

**Examples:**

```questdb-sql title="Calculate the aggregate traveled distance for each car_id"
SELECT car_id, haversine_dist_deg(lat, lon, k)
  FROM table rides
```

## ksum

`ksum(value)` - adds values ignoring missing data (e.g `null` values). Values
are added using the

[Kahan compensated sum algorithm](https://en.wikipedia.org/wiki/Kahan_summation_algorithm).
This is only beneficial for floating-point values such as `float` or `double`.

**Arguments:**

- `value` is any numeric value.

**Return value:**

Return value type is the same as the type of the argument.

**Examples:**

```questdb-sql
SELECT ksum(a)
FROM (SELECT rnd_double() a FROM long_sequence(100));
```

| ksum              |
| :---------------- |
| 52.79143968514029 |

## max

`max(value)` - returns the highest value ignoring missing data (e.g `null`
values).

**Arguments:**

- `value` is any numeric value

**Return value:**

Return value type is the same as the type of the argument.

**Examples:**

```questdb-sql title="Highest transaction amount"
SELECT max(amount) FROM transactions;
```

| max  |
| :--- |
| 55.3 |

```questdb-sql title="Highest transaction amount by payment_type"
SELECT payment_type, max(amount) FROM transactions;
```

| cash_or_card | amount |
| :----------- | :----- |
| cash         | 31.5   |
| card         | 55.3   |
| null         | 29.2   |

## min

`min(value)` - returns the lowest value ignoring missing data (e.g `null`
values).

**Arguments:**

- `value` is any numeric value

**Return value:**

Return value type is the same as the type of the argument.

**Examples:**

```questdb-sql title="Lowest transaction amount"
SELECT min(amount) FROM transactions;
```

| min  |
| :--- |
| 12.5 |

```questdb-sql title="Lowest transaction amount, by payment_type"
SELECT payment_type, min(amount) FROM transactions;
```

| cash_or_card | min  |
| :----------- | :--- |
| cash         | 12.5 |
| card         | 15.3 |
| null         | 22.2 |

## nsum

`nsum(value)` - adds values ignoring missing data (e.g `null` values). Values
are added using the
[Neumaier sum algorithm](https://en.wikipedia.org/wiki/Kahan_summation_algorithm#Further_enhancements).
This is only beneficial for floating-point values such as `float` or `double`.

**Arguments:**

- `value` is any numeric value.

**Return value:**

Return value type is `double`.

**Examples:**

```questdb-sql
SELECT nsum(a)
FROM (SELECT rnd_double() a FROM long_sequence(100));
```

| nsum             |
| :--------------- |
| 49.5442334742831 |

## stddev_samp

`stddev_samp(value)` - calculates the sample standard deviation of values
ignoring missing data (e.g `null` values).

**Arguments:**

- `value` is any numeric value.

**Return value:**

Return value type is `double`.

**Examples:**

```questdb-sql
SELECT stddev_samp(x)
FROM (SELECT x FROM long_sequence(100));
```

| stddev_samp     |
| :-------------- |
| 29.011491975882 |

## row_number

`row_number() over(partition by [partition_expression1, partition_expression2], order by sort_expression1 [ASC | DESC]) as row_num` - returns the sequential number of a row within a partition of a result set, starting at number 1 for first row in each partition.

**Arguments:**
- `partition by` is a clause that divides the result set into partitions(group of rows) .
- `order by` is a clause defines the logical order of the rows within each partition of the result set.

:::note

`partition by` clause is optional. If skipped then row_number() function will conside the result to be a single partition.

:::

**Return value:**

Return value type is `bigint`.

**Examples:**
```questdb-sql title="Display using person name in sequential order"
SELECT person_name,product,amount,
   row_number() over(order by person_name) as row_num
FROM table
```
| row_num | person_name | product | amount |
| :------ | :---------- | :------ | :----- |
| 1.      | Andrew      | Oranges | 50     |
| 2.      | Mark        | Apples  | 25     |
| 3.      | Shannel     | Bananas | 42     |


## sum

`sum(value)` - adds values ignoring missing data (e.g `null` values).

**Arguments:**

- `value` is any numeric value.

**Return value:**

Return value type is the same as the type of the argument.

**Examples:**

```questdb-sql title="Sum all quantities in the transactions table"
SELECT sum(quantity) FROM transactions;
```

| sum |
| :-- |
| 100 |

```questdb-sql title="Sum all quantities in the transactions table, aggregated by item"
SELECT item, sum(quantity) FROM transactions;
```

| item   | count |
| :----- | :---- |
| apple  | 53    |
| orange | 47    |

### Overflow

`sum` does not perform overflow check. To avoid overflow, you can cast the
argument to wider type.

```questdb-sql title="Cast as long to avoid overflow"
SELECT sum(cast(a AS LONG)) FROM table;
```

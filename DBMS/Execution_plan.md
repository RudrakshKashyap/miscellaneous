```
1. FROM        — pick the source table(s), do the JOINs
2. WHERE       — filter individual rows, BEFORE any grouping happens
3. GROUP BY    — bucket the surviving rows into groups
4. aggregates  — compute one value per group (sum, count, argMax, ...)
5. HAVING      — filter groups, using the aggregated values
6. SELECT      — pick/compute the final output columns
7. ORDER BY    — sort the result rows
8. LIMIT       — cut it down

```

- GROUP BY partitions rows — it just tags/sorts them by key, nothing is discarded or reduced yet. Every row that shares a key stays fully intact, sitting together in its bucket.

- so GROUP BY just put all the row in same bucket and lable the bucket "key"

- in later steps like SELECT you can only see the keys of the bucket, but if you want to run some logic/computation inside all rows that are in that bucket you use aggregation function(they can access all rows in buckets).

- so a simple query like (SELECT city from marketplace.warehouses group by city), will produce exactly one row per city because city is the "key/lable" of the bucket

- WHERE can't reference aggregates (WHERE count(\*) > 5 is illegal) — because at step 2, aggregates haven't been computed yet. That's what HAVING is for; it runs at step 5, after aggregation.

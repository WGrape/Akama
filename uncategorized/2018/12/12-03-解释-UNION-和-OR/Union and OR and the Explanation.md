## Union and OR and the Explanation

Two obvious solutions:


```sql
#OR
SELECT name, population, area
FROM World
WHERE area > 3000000 OR population > 25000000
```

And Faster Union

```
#Union
SELECT name, population, area
FROM World
WHERE area > 3000000 

UNION

SELECT name, population, area
FROM World
WHERE population > 25000000
```

Why Union is faster than OR?

Strictly speaking, Using UNION is faster when it comes to cases like scan two different column like this.

(Of course using UNION ALL is much faster than UNION since we don't need to sort the result. But it violates the requirements)

Suppose we are searching population and area, Given that MySQL usually uses one one index per table in a given query, so when it uses the 1st index rather than 2nd index, it would still have to do a table-scan to find rows that fit the 2nd index.

When using UNION, each sub-query can use the index of its search, then combine the sub-query by UNION.

I quote from a benchmark about UNION and OR, feel free to check it out:

```
Scenario 3: Selecting all columns for different fields
            CPU      Reads        Duration       Row Counts
OR           47       1278           443           1228
UNION        31       1334           400           1228

Scenario 4: Selecting Clustered index columns for different fields
            CPU      Reads        Duration       Row Counts
OR           0         319           366           1228
UNION        0          50           193           1228
```

## Comments

Union not always faster than or!
Most good DBMSs use an internal query optimizer to combine the SELECT statements
before they are even processed. In theory, this means that from a performance
perspective, there should be no real difference between using multiple WHERE clause
conditions or a UNION. I say in theory, because, in practice, most query optimizers
don’t always do as good a job as they should. Your best bet is to test both methods to see which will work best for you.
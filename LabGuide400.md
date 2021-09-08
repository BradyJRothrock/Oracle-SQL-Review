## Part 4 - Sorting and Advanced Functions

Here we will talk about how to control the way our results are produced and how to perform advanced functions to return unique results. Answers can be found to questions in a brute force way or by extracting data to other tools, but you'll learn here how perform all these steps in SQL.

## Sorting

We've seen in the Web SQL Developer tool how we can sort data through clicking columns. However, a majority of SQL based tools do not have this funcitonality, or may only sort the sample returned and not from a full data set. To handle this we are going to introduce the **Order By** clause. Here we can provide the column(s) we would like to sort by and declare if we want to sort in **Ascending** or **Descending** order.

Similar to the **Select** statement in the **Order By** we simply provide columns separated by commas. The default is to sort in **Ascending** order but if we want to sort in **Descending** order we simply add **Desc** after the field name.


Here we sort by **TIME ID ASCENDING** and then **CUST ID DESCENDING**.

```SQL
-- Order By Fields
SELECT 
    *
FROM
    SH.SALES
ORDER BY
    TIME_ID, CUST_ID DESC;
```

Next we'll sort by **CALENDAR YEAR ASCENDING** in chronological order.

```SQL
-- Order By With Aggregations
SELECT 
    T.CALENDAR_YEAR AS "YEAR",
	SUM(S.AMOUNT_SOLD) AS "TOTAL SALES"
FROM 
    SH.SALES S
JOIN 
    SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
GROUP BY
    T.CALENDAR_YEAR
ORDER BY
    T.CALENDAR_YEAR;
```

Lastly, we can sort by an aggregated field like the **SUM AMOUNT SOLD**. Here we need to include the aggregate function, not the aliased name.

```SQL
-- Order By Aggregations
SELECT 
    T.CALENDAR_YEAR AS "YEAR",
	SUM(S.AMOUNT_SOLD) AS "TOTAL SALES"
FROM 
    SH.SALES S
JOIN 
    SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
GROUP BY
    T.CALENDAR_YEAR
ORDER BY
    SUM(S.AMOUNT_SOLD);
```

## Aggregate Conditionals

The **Where** clause is very handy when needing to specify a condition for results, however, it does not work for aggregations. If we wanted to return only years where the **SUM AMOUNT SOLD** is greater than 24,000,000 using **Where** will return an error.

```SQL
SELECT 
	T.CALENDAR_YEAR,
    SUM(S.AMOUNT_SOLD) AS SALES
FROM SH.SALES S
	JOIN SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
WHERE
    SUM(S.AMOUNT_SOLD) > 24000000
GROUP BY 
    T.CALENDAR_YEAR;
```

Instead we need a new clause called **Having**. With this we can now specify conditions based on an aggregate calcualtion.

```SQL
SELECT 
	T.CALENDAR_YEAR,
	SUM(S.AMOUNT_SOLD) AS SALES
FROM SH.SALES S
	JOIN SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
GROUP BY 
    T.CALENDAR_YEAR
HAVING SUM(S.AMOUNT_SOLD) > 24000000;
```


## Sub Queries & Pivoting

To begin we need to discuss what a **Sub Query** is and how to use one. A **Sub Query** is a query used in a **From** or **Join** statement in a set of parenthesis.

The simplest example of a **Sub Query** is the following:

```SQL
SELECT
    *
FROM (
    SELECT 
	    T.CALENDAR_YEAR,
	    SUM(S.AMOUNT_SOLD) AS SALES
    FROM SH.SALES S
	    JOIN SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
    GROUP BY 
        T.CALENDAR_YEAR
);
```

Notice how the query returns the exact same table from our previous query. Instead of using **Having** we could use a **Where** statement with our **Sub Query** to return years with greater than 24000000 in sales.

```SQL
SELECT
    *
FROM (
    SELECT 
	    T.CALENDAR_YEAR,
	    SUM(S.AMOUNT_SOLD) AS SALES
    FROM SH.SALES S
	    JOIN SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
    GROUP BY 
        T.CALENDAR_YEAR
)
WHERE SALES > 24000000;
```

We can also replace the `*` with specific field names from the **Sub Query**.

Now we can use a **Sub Query** to help perform a **Pivot** on our dataset. Just like a pivot table in Excel the **Pivot** function allows us to not only calculate aggregate values, but to spread the values across unique columns. The term to describe this is cross-tabulate.

Following the **Sub Query** we use the keyword **Pivot**. Inside the parenthesis we need to provide 3 things. 

- First we need an aggregation to be performed in each cell.
- Second weuse **FOR** to delclare which field with be spread accross columns.
- Lastly we use **IN** to provide a list of columns from the **FOR** field. 

In this example we've added the field **CALENDAR MONTH NUMBER** to use in our **Pivot**. The following query will return the **SUM** of **AMOUNT SOLD** for January, February, & March of each **CALENDAR YEAR**. Notice how we don't need a **GROUP BY** statement because the **PIVOT** performs the aggregation for us. 

```SQL
SELECT *
FROM (
    SELECT
        T.CALENDAR_YEAR,
        T.CALENDAR_MONTH_NUMBER,
        S.AMOUNT_SOLD
    FROM 
        SH.SALES S
        JOIN SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
    )
PIVOT(
    SUM("AMOUNT_SOLD")
    FOR CALENDAR_MONTH_NUMBER IN (1,2,3)
);
```

## Analytic Functions

Analytic functions are row based calculations that can provide valuable insights and comparisons. These analytic functions leverage the same structural approach but each have their own function arguments.

### Structure

The common structure referenced is: `FUNCTION() OVER (<PARTITION BY> <ORDER BY> )`

Although we will change the **FUNCTION()** to perform the different actions, the **PARTITION BY** and **ORDER BY** can be the same or different based on the desired query.

**PARTITION BY** is similar to **GROUP BY** in the fact that it looks at unique combinations of the partition to perform the fucntion. The **ORDER BY** then as expected determines the order of the partitioned values when performing the function. Both are very important when trying to return specific results.

### Rank

**Rank** allows us to assign a numeric increasing value to each row in the order specified. The use of the **Partition** tells SQL when the count starts over and returns to **1** for the next **Partition**.

```SQL
SELECT 
    T.CALENDAR_YEAR AS "YEAR",
    T.CALENDAR_MONTH_NUMBER AS "MONTH",
	SUM(S.AMOUNT_SOLD) AS "TOTAL SALES",
    RANK() OVER(PARTITION BY T.CALENDAR_YEAR ORDER BY SUM(S.AMOUNT_SOLD) DESC) AS "RANK"
FROM 
    SH.SALES S
JOIN 
    SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
GROUP BY
    T.CALENDAR_YEAR,
    T.CALENDAR_MONTH_NUMBER;
```


### Dense Rank

**Dense Rank** is similar to **Rank** except that it allows for ties when the **ORDER BY** value is the same.

```SQL
SELECT
    PROMO_SUBCATEGORY,
    COUNT(*),
    DENSE_RANK() OVER (ORDER BY COUNT(*) ASC) AS "RANK"
FROM 
    SH.PROMOTIONS
GROUP BY
    PROMO_SUBCATEGORY;
```

### Lead & Lag

**Lead** and **Lag** are unique in the fact that they let us look forwards or backwards (i.e. down or up) in a table to retrieve or utilize another rows value. 

**LAG()** allows us to look at a previous record. The function takes 3 arguments. 

- The first is the field or group by aggregation that is being carried forward.
- The second is the offest number of rows to look back.
- And last is the default value when there is no preivous row.

The following query returns for each **CALENDAR YEAR** the **TOTAL SALES** and the previous years sales called **PY SALES**.


```SQL

SELECT
    T.CALENDAR_YEAR,
    SUM(S.AMOUNT_SOLD) AS "TOTAL SALES",
    LAG(SUM(S.AMOUNT_SOLD), 1, 0) OVER (ORDER BY T.CALENDAR_YEAR ASC) AS "PY SALES"
FROM 
    SH.SALES S
    JOIN SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
GROUP BY
    T.CALENDAR_YEAR;
    
```


**LEAD()** allows us to look at a future record. The function takes 3 arguments. 

- The first is the field or group by aggregation that is being carried forward.
- The second is the offest number of rows to look forward.
- And last is the default value when there is no future row.

The following query returns for each **CALENDAR YEAR** the **TOTAL SALES** and the next years sales called **FY SALES**.


```SQL

SELECT
    T.CALENDAR_YEAR,
    SUM(S.AMOUNT_SOLD) AS "TOTAL SALES",
    LEAD(SUM(S.AMOUNT_SOLD), 1, 0) OVER (ORDER BY T.CALENDAR_YEAR ASC) AS "FY SALES"
FROM 
    SH.SALES S
    JOIN SH.TIMES T ON (S.TIME_ID = T.TIME_ID)
GROUP BY
    T.CALENDAR_YEAR;
    
```


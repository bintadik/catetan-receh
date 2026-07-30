# SQL Window Functions — Complete Guide

A window function performs a calculation across a set of rows related to the current row, **without collapsing rows into a single output** (unlike `GROUP BY`). This makes them ideal for running totals, rankings, moving averages, and row-to-row comparisons while keeping every row visible.

---

## 1. Core Syntax

```sql
function_name(expression) OVER (
    [PARTITION BY column1, column2, ...]
    [ORDER BY column3, column4, ...]
    [ROWS | RANGE BETWEEN frame_start AND frame_end]
)
```

| Clause         | Purpose                                                                      |
| -------------- | ---------------------------------------------------------------------------- |
| `PARTITION BY` | Splits rows into groups (like `GROUP BY`, but rows aren't collapsed)         |
| `ORDER BY`     | Defines the order within each partition (required for ranking/running calcs) |
| `ROWS`/`RANGE` | Defines the "frame" — which rows around the current row are included         |

**Key difference from `GROUP BY`:** `GROUP BY` returns one row per group. Window functions return **one row per input row**, with the calculated value attached.

---

## 2. Setup Example Table

We'll use this `sales` table throughout:

```sql
CREATE TABLE sales (
    id INT,
    salesperson VARCHAR(50),
    region VARCHAR(50),
    sale_date DATE,
    amount DECIMAL(10,2)
);
```

| id  | salesperson | region | sale_date  | amount |
| --- | ----------- | ------ | ---------- | ------ |
| 1   | Alice       | East   | 2024-01-05 | 200    |
| 2   | Bob         | East   | 2024-01-06 | 150    |
| 3   | Alice       | East   | 2024-01-10 | 300    |
| 4   | Carol       | West   | 2024-01-07 | 400    |
| 5   | Dave        | West   | 2024-01-09 | 250    |
| 6   | Carol       | West   | 2024-01-15 | 100    |

---

## 3. Ranking Functions

### 3.1 `ROW_NUMBER()`
Assigns a unique, sequential number to each row within a partition — no ties.

```sql
SELECT
    salesperson,
    region,
    amount,
    ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC) AS rn
FROM sales;
```

### 3.2 `RANK()`
Same rank for ties, but **skips** the next rank number(s).

```sql
SELECT
    salesperson,
    amount,
    RANK() OVER (ORDER BY amount DESC) AS rnk
FROM sales;
```
> Example: ranks 1, 2, 2, 4 (rank 3 is skipped after a tie)

### 3.3 `DENSE_RANK()`
Same rank for ties, but **does not skip** the next rank.

```sql
SELECT
    salesperson,
    amount,
    DENSE_RANK() OVER (ORDER BY amount DESC) AS drnk
FROM sales;
```
> Example: ranks 1, 2, 2, 3 (no gap)

### 3.4 `NTILE(n)`
Divides rows in a partition into `n` roughly equal buckets — useful for quartiles/percentiles.

```sql
SELECT
    salesperson,
    amount,
    NTILE(4) OVER (ORDER BY amount DESC) AS quartile
FROM sales;
```

**Cheat sheet:**

| Function       | Ties         | Gaps after ties |
| -------------- | ------------ | --------------- |
| `ROW_NUMBER()` | Never ties   | N/A             |
| `RANK()`       | Ties allowed | Skips (gap)     |
| `DENSE_RANK()` | Ties allowed | No gap          |

---

## 4. Offset / Value Functions

### 4.1 `LAG()` — look at a previous row
```sql
SELECT
    salesperson,
    sale_date,
    amount,
    LAG(amount, 1) OVER (PARTITION BY salesperson ORDER BY sale_date) AS prev_amount
FROM sales;
```
Third argument = default value if there's no previous row: `LAG(amount, 1, 0)`.

### 4.2 `LEAD()` — look at a following row
```sql
SELECT
    salesperson,
    sale_date,
    amount,
    LEAD(amount, 1) OVER (PARTITION BY salesperson ORDER BY sale_date) AS next_amount
FROM sales;
```

### 4.3 `FIRST_VALUE()` / `LAST_VALUE()`
```sql
SELECT
    salesperson,
    amount,
    FIRST_VALUE(amount) OVER (PARTITION BY salesperson ORDER BY sale_date) AS first_sale,
    LAST_VALUE(amount) OVER (
        PARTITION BY salesperson ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_sale
FROM sales;
```
> ⚠️ `LAST_VALUE()` needs an explicit frame (`ROWS BETWEEN ... UNBOUNDED FOLLOWING`) — otherwise the default frame stops at the current row and gives wrong results.

### 4.4 `NTH_VALUE()`
```sql
SELECT
    salesperson,
    amount,
    NTH_VALUE(amount, 2) OVER (PARTITION BY salesperson ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS second_sale
FROM sales;
```

---

## 5. Aggregate Functions as Window Functions

Any aggregate (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) becomes a window function when paired with `OVER()`.

### 5.1 Running total
```sql
SELECT
    salesperson,
    sale_date,
    amount,
    SUM(amount) OVER (PARTITION BY salesperson ORDER BY sale_date) AS running_total
FROM sales;
```

### 5.2 Moving average (last 3 rows)
```sql
SELECT
    salesperson,
    sale_date,
    amount,
    AVG(amount) OVER (
        PARTITION BY salesperson
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3
FROM sales;
```

### 5.3 Percent of total
```sql
SELECT
    region,
    salesperson,
    amount,
    amount / SUM(amount) OVER (PARTITION BY region) AS pct_of_region
FROM sales;
```

### 5.4 Count without collapsing rows
```sql
SELECT
    salesperson,
    region,
    COUNT(*) OVER (PARTITION BY region) AS region_sale_count
FROM sales;
```

---

## 6. Frame Clauses — `ROWS` vs `RANGE`

The frame defines exactly *which rows* are included in the calculation relative to the current row.

```sql
... OVER (
    PARTITION BY col
    ORDER BY col2
    ROWS BETWEEN <start> AND <end>
)
```

Common frame boundaries:

| Boundary              | Meaning                         |
| --------------------- | ------------------------------- |
| `UNBOUNDED PRECEDING` | From the start of the partition |
| `N PRECEDING`         | N rows before current row       |
| `CURRENT ROW`         | The current row                 |
| `N FOLLOWING`         | N rows after current row        |
| `UNBOUNDED FOLLOWING` | To the end of the partition     |

**`ROWS`** counts physical rows. **`RANGE`** counts logical value ranges (rows with the same `ORDER BY` value are treated as one peer group) — subtle but important with duplicate values.

Default frame (when `ORDER BY` is present but no frame is specified):
```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

Examples:
```sql
-- Sum of current + 1 row before + 1 row after
SUM(amount) OVER (ORDER BY sale_date ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)

-- Cumulative sum from start to current row (explicit)
SUM(amount) OVER (ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- Sum across the ENTIRE partition (ignores order)
SUM(amount) OVER (PARTITION BY region ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
```

---

## 7. `WINDOW` Clause (reduce repetition)

If you reuse the same window definition multiple times, name it once:

```sql
SELECT
    salesperson,
    amount,
    SUM(amount) OVER w AS running_total,
    AVG(amount) OVER w AS running_avg
FROM sales
WINDOW w AS (PARTITION BY salesperson ORDER BY sale_date);
```

---

## 8. Filtering Window Function Results

You **cannot** use a window function directly in `WHERE` — window functions are evaluated *after* `WHERE`/`GROUP BY`/`HAVING`. Use a subquery or CTE instead.

```sql
-- ❌ This will error:
SELECT salesperson, RANK() OVER (ORDER BY amount DESC) AS rnk
FROM sales
WHERE rnk = 1;

-- ✅ Correct approach:
WITH ranked AS (
    SELECT
        salesperson,
        amount,
        RANK() OVER (ORDER BY amount DESC) AS rnk
    FROM sales
)
SELECT * FROM ranked WHERE rnk = 1;
```

---

## 9. Order of SQL Execution (why window functions behave this way)

```
FROM  →  WHERE  →  GROUP BY  →  HAVING  →  WINDOW FUNCTIONS  →  SELECT  →  DISTINCT  →  ORDER BY  →  LIMIT
```

Window functions run **after** grouping/filtering but **before** the final `SELECT`/`ORDER BY` — this is why they can't be referenced in `WHERE`, but can appear in `SELECT` and (in some databases) `ORDER BY` directly.

---

## 10. Practical Patterns / Recipes

### 10.1 Top N per group
```sql
WITH ranked AS (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC) AS rn
    FROM sales
)
SELECT * FROM ranked WHERE rn <= 2;   -- top 2 sales per region
```

### 10.2 Detect gaps between consecutive events
```sql
SELECT
    salesperson,
    sale_date,
    sale_date - LAG(sale_date) OVER (PARTITION BY salesperson ORDER BY sale_date) AS days_since_last_sale
FROM sales;
```

### 10.3 Running difference from average
```sql
SELECT
    salesperson,
    amount,
    amount - AVG(amount) OVER () AS diff_from_overall_avg
FROM sales;
```

### 10.4 De-duplicate rows (keep latest record per group)
```sql
WITH ranked AS (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY salesperson ORDER BY sale_date DESC) AS rn
    FROM sales
)
SELECT * FROM ranked WHERE rn = 1;
```

### 10.5 Cumulative distribution / percentile
```sql
SELECT
    salesperson,
    amount,
    CUME_DIST() OVER (ORDER BY amount) AS cume_dist,
    PERCENT_RANK() OVER (ORDER BY amount) AS pct_rank
FROM sales;
```

---

## 11. Common Mistakes to Avoid

1. **Forgetting `PARTITION BY`** — without it, the window spans the entire result set.
2. **Using window functions in `WHERE`** — filter with a CTE/subquery instead.
3. **Assuming `LAST_VALUE()` works without a frame** — always specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` when you want the true last value in the partition.
4. **Confusing `RANGE` and `ROWS`** — `RANGE` groups peer rows with equal `ORDER BY` values; `ROWS` treats every row individually.
5. **Overusing `SELECT *` with multiple window functions** — name each output column clearly.

---

## 12. Quick Reference Table

| Function         | Category     | Typical Use                 |
| ---------------- | ------------ | --------------------------- |
| `ROW_NUMBER()`   | Ranking      | Unique sequential numbering |
| `RANK()`         | Ranking      | Ranking with gaps on ties   |
| `DENSE_RANK()`   | Ranking      | Ranking without gaps        |
| `NTILE(n)`       | Ranking      | Bucketing into n groups     |
| `LAG()`          | Offset       | Compare to a previous row   |
| `LEAD()`         | Offset       | Compare to a following row  |
| `FIRST_VALUE()`  | Offset       | First value in the frame    |
| `LAST_VALUE()`   | Offset       | Last value in the frame     |
| `SUM() OVER`     | Aggregate    | Running totals              |
| `AVG() OVER`     | Aggregate    | Moving averages             |
| `COUNT() OVER`   | Aggregate    | Row counts per partition    |
| `PERCENT_RANK()` | Distribution | Relative standing (0 to 1)  |
| `CUME_DIST()`    | Distribution | Cumulative distribution     |

---

## 13. Practice Exercises

Try these against the `sales` table above:

1. Rank salespeople by total amount sold (all regions combined).
2. Find each salesperson's first and most recent sale date.
3. Calculate a 7-day moving sum of sales amount per region.
4. Find the difference between each sale and the previous sale by the same salesperson.
5. Identify the top-selling salesperson in each region using `ROW_NUMBER()`.

---

*Tip: Window function syntax is nearly identical across PostgreSQL, MySQL 8+, SQL Server, and Snowflake. Minor differences exist in `LAST_VALUE()` frame defaults and `NTILE` tie-handling — always check your specific database's docs for edge cases.*
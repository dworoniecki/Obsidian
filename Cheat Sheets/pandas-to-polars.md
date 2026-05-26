# Pandas to Polars Cheatsheet

A practical guide for transitioning from pandas to Polars, with examples using lending/financial data scenarios.

---

## Common Workflows

A complete workflow showing the power of Polars' method chaining:

```python
# Pandas
df = pd.read_parquet('loans.parquet')
df = df[df['loan_status'].isin(['CURRENT', 'PAID_OFF'])]
df['balance_thousands'] = df['balance'] / 1000
df = df.groupby('originator').agg(
    total_balance=('balance', 'sum'),
    loan_count=('loan_id', 'count')
).reset_index()
df = df.sort_values('total_balance', ascending=False)
df = df.head(10)

# Polars
df = (
    pl.read_parquet('loans.parquet')
    .filter(pl.col('loan_status').is_in(['CURRENT', 'PAID_OFF']))
    .with_columns((pl.col('balance') / 1000).alias('balance_thousands'))
    .group_by('originator')
    .agg([
        pl.col('balance').sum().alias('total_balance'),
        pl.col('loan_id').count().alias('loan_count')
    ])
    .sort('total_balance', descending=True)
    .head(10)
)
```

### Expression Chaining in Polars

Polars expressions can be chained and reused efficiently:

```python
# Create reusable expressions
balance_expr = pl.col('balance')
status_filter = pl.col('loan_status').is_in(['CURRENT', 'PAID_OFF'])

# Chain expressions in query
df = (
    df
    .filter(status_filter)
    .group_by('originator')
    .agg([
        balance_expr.sum().alias('total'),
        balance_expr.mean().alias('avg'),
        balance_expr.count().alias('count')
    ])
)

# Aggregate all columns with same operation
df.group_by('originator').agg(pl.all().sum())
```

---

## Setup & Imports

```python
# Pandas
import pandas as pd

# Polars
import polars as pl
```

---

## Reading Data

### CSV Files
```python
# Pandas
df = pd.read_csv('loans.csv')
df = pd.read_csv('loans.csv', parse_dates=['origination_date'])

# Polars
df = pl.read_csv('loans.csv')
df = pl.read_csv('loans.csv', try_parse_dates=True)
df = pl.read_csv('loans.csv', has_header=True)  # Explicitly specify header

# Select specific columns only
# Pandas
df = pd.read_csv('loans.csv', usecols=['loan_id', 'balance', 'originator'])

# Polars
df = pl.read_csv('loans.csv', columns=['loan_id', 'balance', 'originator'])

# Specify data types on read
# Pandas
df = pd.read_csv('loans.csv', dtype={'loan_id': str, 'balance': float, 'zip_code': str})

# Polars
df = pl.read_csv('loans.csv', dtypes={'loan_id': pl.Utf8, 'balance': pl.Float64, 'zip_code': pl.Utf8})

# Or using schema
df = pl.read_csv('loans.csv', schema={'loan_id': pl.Utf8, 'balance': pl.Float64, 'zip_code': pl.Utf8})
```

### Parquet Files
```python
# Pandas
df = pd.read_parquet('loans.parquet')

# Polars
df = pl.read_parquet('loans.parquet')

# Polars - Lazy (doesn't load until needed)
df = pl.scan_parquet('loans.parquet')  # Returns LazyFrame
df = df.collect()  # Execute and load

# Select specific columns only
# Pandas
df = pd.read_parquet('loans.parquet', columns=['loan_id', 'balance', 'originator'])

# Polars
df = pl.read_parquet('loans.parquet', columns=['loan_id', 'balance', 'originator'])

# Polars - Lazy with column selection (more efficient - only reads needed columns)
df = pl.scan_parquet('loans.parquet').select(['loan_id', 'balance', 'originator']).collect()

# Override data types on read (less common with parquet since types are stored)
# Pandas
df = pd.read_parquet('loans.parquet')
df['loan_id'] = df['loan_id'].astype(str)

# Polars - cast after reading
df = pl.read_parquet('loans.parquet').with_columns(pl.col('loan_id').cast(pl.Utf8))
```

### Multiple Files (Glob Pattern)
```python
# Pandas
import glob
dfs = [pd.read_parquet(f) for f in glob.glob('data/*.parquet')]
df = pd.concat(dfs, ignore_index=True)

# Polars
df = pl.read_parquet('data/*.parquet')

# Polars - Lazy (better for large datasets)
df = pl.scan_parquet('data/*.parquet').collect()
```

---

## Basic Inspection

| Method | Pandas | Polars | Description | Common Parameters |
|--------|--------|--------|-------------|-------------------|
| **View first rows** | `df.head()` | `df.head()` | Display first N rows | `n=5` (default 5 for both) |
| **View last rows** | `df.tail()` | `df.tail()` | Display last N rows | `n=5` (default 5 for both) |
| **Dimensions** | `df.shape` | `df.shape` | Returns tuple of (rows, columns) | N/A (property, not method) |
| **Column names** | `df.columns` | `df.columns` | Returns list of column names | N/A (property) |
| **Data types** | `df.dtypes` | `df.dtypes` | Show data type of each column | N/A (property) |
| **Quick overview** | `df.info()` | `df.glimpse()` | Summary including dtypes, non-null counts, memory | Pandas: `verbose=True`, `show_counts=True`<br>Polars: `return_as_string=False` |
| **Statistics** | `df.describe()` | `df.describe()` | Summary statistics (mean, std, min, max, etc.) | Pandas: `include='all'`, `percentiles=[.25, .5, .75]`<br>Polars: `percentiles=[0.25, 0.5, 0.75]` |
| **Memory usage** | `df.memory_usage()` | `df.estimated_size()` | Memory consumed by DataFrame | Pandas: `deep=True` for actual usage<br>Polars: returns bytes (single value) |
| **Null counts** | `df.isna().sum()` | `df.null_count()` | Count missing values per column | Polars returns DataFrame with counts |
| **Sample rows** | `df.sample(n=5)` | `df.sample(n=5)` | Randomly select N rows | `n=` (number) or `frac=` (fraction), `random_state=`/`seed=` |

### Examples

```python
# Pandas
df.head(10)                    # First 10 rows
df.info(verbose=True)          # Detailed info
df.describe(include='all')     # Stats for all columns
df.memory_usage(deep=True)     # Actual memory usage
df.sample(frac=0.1)            # Random 10% sample

# Polars
df.head(10)                    # First 10 rows
df.glimpse()                   # Detailed info
df.describe()                  # Stats for all columns
df.estimated_size('mb')        # Memory in megabytes
df.sample(fraction=0.1)        # Random 10% sample
```

---

## Selecting Columns

```python
# Pandas
df['loan_id']
df[['loan_id', 'balance', 'originator']]

# Polars
df['loan_id']  # Returns Series
df.select('loan_id')  # Returns DataFrame
df.select(['loan_id', 'balance', 'originator'])
df.select(pl.col('loan_id', 'balance', 'originator'))

# Select by pattern
# Pandas
df.filter(regex='^loan_')

# Polars
df.select(pl.col('^loan_.*$'))

# Select columns by position (bracket notation)
# Pandas
df.iloc[:, [0, 2, 3]]  # Columns at index 0, 2, 3

# Polars
df[:, [0, 2, 3]]  # Same - columns at index 0, 2, 3
```

---

## Subsetting Rows and Columns Together

```python
# Select specific rows by position
# Pandas
df.iloc[2:4]  # Rows 2-3 (4 is exclusive)

# Polars
df[2:4]  # Rows 2-3 (4 is exclusive)
df[2:4, :]  # More explicit - rows 2-3, all columns

# Select rows by position AND columns by position
# Pandas
df.iloc[2:4, [1, 3]]  # Rows 2-3, columns at position 1 and 3

# Polars
df[2:4, [1, 3]]  # Rows 2-3, columns at position 1 and 3

# Select rows by condition AND specific columns
# Pandas
df.loc[df['balance'] > 10000, ['loan_id', 'originator']]

# Polars
df.filter(pl.col('balance') > 10000).select(['loan_id', 'originator'])
# Or using bracket notation
df[df['balance'] > 10000, ['loan_id', 'originator']]
```

---

## Filtering Rows

```python
# Pandas
df[df['balance'] > 10000]
df[df['loan_status'] == 'CURRENT']
df[(df['balance'] > 10000) & (df['loan_status'] == 'CURRENT')]
df[df['originator'].isin(['ACME', 'ZENITH'])]

# Polars
df.filter(pl.col('balance') > 10000)
df.filter(pl.col('loan_status') == 'CURRENT')
df.filter((pl.col('balance') > 10000) & (pl.col('loan_status') == 'CURRENT'))
df.filter(pl.col('originator').is_in(['ACME', 'ZENITH']))
```

---

## Adding/Modifying Columns

```python
# Pandas
df['balance_usd'] = df['balance'] / 100
df['loan_age_days'] = (pd.Timestamp.now() - df['origination_date']).dt.days

# Polars
df = df.with_columns(
    (pl.col('balance') / 100).alias('balance_usd')
)
df = df.with_columns(
    (pl.lit(pl.datetime.now()) - pl.col('origination_date')).dt.total_days().alias('loan_age_days')
)

# Multiple columns at once
# Pandas
df['balance_usd'] = df['balance'] / 100
df['balance_thousands'] = df['balance'] / 1000

# Polars
df = df.with_columns([
    (pl.col('balance') / 100).alias('balance_usd'),
    (pl.col('balance') / 1000).alias('balance_thousands')
])

# Add row number/index column
# Pandas
df['row_num'] = range(len(df))
df = df.reset_index(drop=False, names='row_num')

# Polars
df = df.with_row_count(name='row_num')  # Adds column starting at 0
```

---

## Renaming Columns

```python
# Pandas
df.rename(columns={'old_name': 'new_name'})
df.columns = ['loan_id', 'balance', 'status']

# Polars
df.rename({'old_name': 'new_name'})
df.columns = ['loan_id', 'balance', 'status']  # Works in Polars too

# Rename all columns with a function
# Pandas
df.columns = df.columns.str.lower()

# Polars
df.columns = [col.lower() for col in df.columns]
```

---

## Sorting

```python
# Pandas
df.sort_values('balance')
df.sort_values('balance', ascending=False)
df.sort_values(['originator', 'balance'], ascending=[True, False])

# Polars
df.sort('balance')
df.sort('balance', descending=True)
df.sort(['originator', 'balance'], descending=[False, True])
```

---

## Groupby & Aggregations

### Basic Aggregations
```python
# Pandas
df.groupby('originator')['balance'].sum()
df.groupby('originator')['balance'].agg(['sum', 'mean', 'count'])

# Polars - two syntax styles (both work)
# Style 1: Using pl.col()
df.group_by('originator').agg(pl.col('balance').sum())

# Style 2: Using pl.sum() directly
df.group_by('originator').agg(pl.sum('balance'))

# Multiple aggregations
df.group_by('originator').agg([
    pl.col('balance').sum().alias('total_balance'),
    pl.col('balance').mean().alias('avg_balance'),
    pl.col('balance').count().alias('loan_count')
])

# Or with shorthand syntax
df.group_by('originator').agg([
    pl.sum('balance').alias('total_balance'),
    pl.mean('balance').alias('avg_balance'),
    pl.count('balance').alias('loan_count')
])
```

### All Available Aggregation Functions
```python
# Polars aggregation functions (can use pl.col('column').func() or pl.func('column'))
df.select([
    pl.sum('balance').alias('sum'),
    pl.min('balance').alias('min'),
    pl.max('balance').alias('max'),
    pl.mean('balance').alias('mean'),
    pl.median('balance').alias('median'),
    pl.std('balance').alias('std'),
    pl.var('balance').alias('variance'),
    pl.count('balance').alias('count'),
    pl.first('balance').alias('first'),
    pl.last('balance').alias('last'),
    pl.quantile('balance', 0.75).alias('q75'),
    # All columns aggregation
    pl.all().sum()  # Apply sum to all numeric columns
])
```

### Multiple Groupby Columns
```python
# Pandas
df.groupby(['originator', 'loan_status'])['balance'].sum()

# Polars
df.group_by(['originator', 'loan_status']).agg(
    pl.col('balance').sum()
)
```

### Named Aggregations
```python
# Pandas
df.groupby('originator').agg(
    total_balance=('balance', 'sum'),
    avg_balance=('balance', 'mean'),
    loan_count=('loan_id', 'count')
)

# Polars
df.group_by('originator').agg([
    pl.col('balance').sum().alias('total_balance'),
    pl.col('balance').mean().alias('avg_balance'),
    pl.col('loan_id').count().alias('loan_count')
])
```

### Aggregate All Columns
```python
# Pandas
df.groupby('originator').sum()

# Polars - aggregate all numeric columns
df.group_by('originator').agg(pl.all().sum())

# Polars - multiple aggregations on all columns
df.group_by('originator').agg([
    pl.all().sum(),
    pl.all().mean()
])
```

### Apply Custom Function to Groups
```python
# Pandas
df.groupby('originator')['loan_id'].apply(lambda x: x.sample(1))

# Polars
df.group_by('originator').agg(
    pl.col('loan_id').apply(lambda group: group[0])  # Get first item
)

# Or for sampling (better approach)
df.group_by('originator').agg(
    pl.col('loan_id').first()
)
```

---

## Joins

```python
# Pandas
pd.merge(loans, payments, on='loan_id', how='left')
pd.merge(loans, payments, left_on='loan_id', right_on='id', how='inner')

# Polars
loans.join(payments, on='loan_id', how='left')
loans.join(payments, left_on='loan_id', right_on='id', how='inner')

# Additional join types in Polars
loans.join(payments, on='loan_id', how='outer')  # Full outer join
loans.join(payments, on='loan_id', how='anti')   # Anti join - rows in left NOT in right
loans.join(payments, on='loan_id', how='semi')   # Semi join - rows in left that ARE in right
```

---

## Handling Missing Values

```python
# Pandas
df.isna()
df.isna().sum()
df.dropna()
df.dropna(subset=['loan_id'])
df.fillna(0)
df['balance'].fillna(df['balance'].mean())

# Polars
df.null_count()  # Count nulls per column
df.drop_nulls()
df.drop_nulls(subset=['loan_id'])
df.fill_null(0)
df.with_columns(
    pl.col('balance').fill_null(pl.col('balance').mean())
)

# Fill null with different strategies
# Pandas
df['balance'].fillna(method='ffill')  # Forward fill
df['balance'].fillna(method='bfill')  # Backward fill

# Polars - various fill strategies
df.fill_null(strategy='forward')   # Forward fill
df.fill_null(strategy='backward')  # Backward fill
df.fill_null(strategy='min')       # Fill with column minimum
df.fill_null(strategy='max')       # Fill with column maximum
df.fill_null(strategy='mean')      # Fill with column mean
df.fill_null(strategy='zero')      # Fill with 0
df.fill_null(strategy='one')       # Fill with 1

# Fill NaN (floating point NaN, different from NULL)
# Pandas
df.fillna(0)  # Handles both NaN and None

# Polars
df.fill_nan(0)  # Only for floating point NaN values
df.fill_null(0)  # Only for NULL values

# To handle both in Polars
df.fill_nan(0).fill_null(0)
```

---

## String Operations

```python
# Pandas
df['originator'].str.lower()
df['originator'].str.upper()
df['originator'].str.strip()
df['originator'].str.contains('ACME')
df['originator'].str.replace('_', '-')
df['originator'].str.len()

# Polars
df.select(pl.col('originator').str.to_lowercase())
df.select(pl.col('originator').str.to_uppercase())
df.select(pl.col('originator').str.strip_chars())
df.filter(pl.col('originator').str.contains('ACME'))
df.select(pl.col('originator').str.replace('_', '-'))
df.select(pl.col('originator').str.lengths())  # or .str.len_chars()
```

---

## Date Operations

```python
# Pandas
df['origination_date'].dt.year
df['origination_date'].dt.month
df['origination_date'].dt.day
df['origination_date'].dt.dayofweek
df['origination_date'] + pd.Timedelta(days=30)

# Polars
df.select(pl.col('origination_date').dt.year())
df.select(pl.col('origination_date').dt.month())
df.select(pl.col('origination_date').dt.day())
df.select(pl.col('origination_date').dt.weekday())
df.select(pl.col('origination_date') + pl.duration(days=30))
```

---

## Window Functions

### Ranking
```python
# Pandas
df['rank'] = df.groupby('originator')['balance'].rank(ascending=False)

# Polars
df = df.with_columns(
    pl.col('balance').rank(descending=True).over('originator').alias('rank')
)
```

### Cumulative Sum
```python
# Pandas
df['cumsum'] = df.groupby('originator')['balance'].cumsum()

# Polars
df = df.with_columns(
    pl.col('balance').cum_sum().over('originator').alias('cumsum')
)
```

### Shift (Lag/Lead)
```python
# Pandas - Shift down (lag - get previous value)
df['prev_balance'] = df.groupby('originator')['balance'].shift(1)

# Pandas - Shift up (lead - get next value)
df['next_balance'] = df.groupby('originator')['balance'].shift(-1)

# Polars - Shift down (lag - get previous value)
df = df.with_columns(
    pl.col('balance').shift(1).over('originator').alias('prev_balance')
)

# Polars - Shift up (lead - get next value)
df = df.with_columns(
    pl.col('balance').shift(-1).over('originator').alias('next_balance')
)

# Example: Calculate payment amount change from previous month
# Pandas
df = df.sort_values(['loan_id', 'payment_date'])
df['payment_change'] = df.groupby('loan_id')['payment_amount'].diff()

# Polars
df = df.sort(['loan_id', 'payment_date'])
df = df.with_columns(
    (pl.col('payment_amount') - pl.col('payment_amount').shift(1))
    .over('loan_id')
    .alias('payment_change')
)
```

### Rolling Operations
```python
# Pandas
df['rolling_avg'] = df['balance'].rolling(window=3).mean()
df['rolling_sum'] = df['balance'].rolling(window=3).sum()
df['rolling_std'] = df['balance'].rolling(window=3).std()

# Polars - comprehensive rolling functions
df = df.with_columns([
    # Rolling mean
    pl.col('balance').rolling_mean(window_size=3).alias('rolling_mean'),
    
    # Rolling sum
    pl.col('balance').rolling_sum(window_size=3).alias('rolling_sum'),
    
    # Rolling min/max
    pl.col('balance').rolling_min(window_size=3).alias('rolling_min'),
    pl.col('balance').rolling_max(window_size=3).alias('rolling_max'),
    
    # Rolling standard deviation
    pl.col('balance').rolling_std(window_size=3).alias('rolling_std'),
    
    # Rolling variance
    pl.col('balance').rolling_var(window_size=3).alias('rolling_var'),
    
    # Rolling median (requires min_periods)
    pl.col('balance').rolling_median(window_size=3, min_periods=1).alias('rolling_median'),
    
    # Rolling quantile
    pl.col('balance').rolling_quantile(
        quantile=0.75, 
        window_size=3, 
        min_periods=1
    ).alias('rolling_q75'),
    
    # Rolling skew
    pl.col('balance').rolling_skew(window_size=3).alias('rolling_skew'),
])

# Rolling with custom function (using apply)
import numpy as np
df = df.with_columns(
    pl.col('balance').rolling_apply(
        function=np.nanstd, 
        window_size=3
    ).alias('rolling_custom')
)
```

---

## GroupBy Transform

Transform allows you to apply aggregations but keep the original DataFrame shape (broadcasting the aggregate value to all rows in each group).

### Basic Transform
```python
# Example: Add total balance per originator to each row
# Pandas
df['originator_total_balance'] = df.groupby('originator')['balance'].transform('sum')

# Polars (using .over() - same as window function)
df = df.with_columns(
    pl.col('balance').sum().over('originator').alias('originator_total_balance')
)
```

### Multiple Aggregations with Transform
```python
# Example: Add multiple originator-level stats to each loan row
# Pandas
df['originator_total'] = df.groupby('originator')['balance'].transform('sum')
df['originator_avg'] = df.groupby('originator')['balance'].transform('mean')
df['originator_count'] = df.groupby('originator')['loan_id'].transform('count')

# Polars
df = df.with_columns([
    pl.col('balance').sum().over('originator').alias('originator_total'),
    pl.col('balance').mean().over('originator').alias('originator_avg'),
    pl.col('loan_id').count().over('originator').alias('originator_count')
])
```

### Window Functions Over Multiple Columns
```python
# Example: Calculate totals grouped by multiple columns simultaneously
# Pandas
df['sum_by_originator'] = df.groupby('originator')['balance'].transform('sum')
df['sum_by_status'] = df.groupby('loan_status')['balance'].transform('sum')

# Polars - can calculate multiple groupings in one operation
df = df.with_columns([
    pl.col('balance').sum().over('originator').alias('sum_by_originator'),
    pl.col('balance').sum().over('loan_status').alias('sum_by_status'),
    pl.col('balance').sum().over(['originator', 'loan_status']).alias('sum_by_both')
])
```

### Calculate Percentage of Group Total
```python
# Example: Each loan's percentage of originator's total balance
# Pandas
df['pct_of_originator_total'] = (
    df['balance'] / df.groupby('originator')['balance'].transform('sum') * 100
)

# Polars
df = df.with_columns(
    (pl.col('balance') / pl.col('balance').sum().over('originator') * 100)
    .alias('pct_of_originator_total')
)
```

### Custom Function with Transform
```python
# Example: Z-score within each originator (standardize)
# Pandas
df['balance_zscore'] = df.groupby('originator')['balance'].transform(
    lambda x: (x - x.mean()) / x.std()
)

# Polars
df = df.with_columns(
    ((pl.col('balance') - pl.col('balance').mean().over('originator')) / 
     pl.col('balance').std().over('originator'))
    .alias('balance_zscore')
)
```

### Fillna with Group Mean
```python
# Example: Fill missing balances with originator's average
# Pandas
df['balance'] = df['balance'].fillna(
    df.groupby('originator')['balance'].transform('mean')
)

# Polars
df = df.with_columns(
    pl.col('balance').fill_null(
        pl.col('balance').mean().over('originator')
    )
)
```

---

## Unique Values & Counts

```python
# Pandas
df['originator'].unique()
df['originator'].nunique()
df['originator'].value_counts()

# Polars
df['originator'].unique()
df['originator'].n_unique()
df['originator'].value_counts()
```

---

## Conditional Logic (if/else)

```python
# Pandas
df['risk_tier'] = df['balance'].apply(
    lambda x: 'High' if x > 50000 else 'Medium' if x > 10000 else 'Low'
)

# Or with numpy
import numpy as np
df['risk_tier'] = np.select(
    [df['balance'] > 50000, df['balance'] > 10000],
    ['High', 'Medium'],
    default='Low'
)

# Polars
df = df.with_columns(
    pl.when(pl.col('balance') > 50000)
    .then(pl.lit('High'))
    .when(pl.col('balance') > 10000)
    .then(pl.lit('Medium'))
    .otherwise(pl.lit('Low'))
    .alias('risk_tier')
)
```

---

## Pivot Tables

```python
# Pandas
df.pivot_table(
    values='balance',
    index='originator',
    columns='loan_status',
    aggfunc='sum'
)

# Polars
df.pivot(
    values='balance',
    index='originator',
    columns='loan_status',
    aggregate_function='sum'
)
```

---

## Melt / Unpivot (Wide to Long Format)

Melt is used to transform data from wide format to long format - useful when you receive data with values spread across multiple columns that should be in rows.

### Basic Melt
```python
# Example: Convert monthly payment columns to long format
# Wide format:
# loan_id | jan_payment | feb_payment | mar_payment
# 123     | 500         | 500         | 500

# Pandas
df_long = df.melt(
    id_vars=['loan_id'],
    value_vars=['jan_payment', 'feb_payment', 'mar_payment'],
    var_name='month',
    value_name='payment_amount'
)

# Polars
df_long = df.unpivot(
    index='loan_id',
    on=['jan_payment', 'feb_payment', 'mar_payment'],
    variable_name='month',
    value_name='payment_amount'
)

# Result (long format):
# loan_id | month       | payment_amount
# 123     | jan_payment | 500
# 123     | feb_payment | 500
# 123     | mar_payment | 500
```

### Melt All Columns Except ID Columns
```python
# Example: Melt all payment columns (don't need to list them all)
# Pandas
df_long = df.melt(
    id_vars=['loan_id', 'originator'],
    var_name='metric',
    value_name='value'
)

# Polars - use column selector to specify which to unpivot
df_long = df.unpivot(
    index=['loan_id', 'originator'],
    variable_name='metric',
    value_name='value'
)
```

### Melt with Pattern Matching
```python
# Example: Melt only columns that match a pattern (e.g., all month columns)
# Pandas
payment_cols = [col for col in df.columns if col.endswith('_payment')]
df_long = df.melt(
    id_vars=['loan_id'],
    value_vars=payment_cols,
    var_name='month',
    value_name='payment_amount'
)

# Polars
df_long = df.unpivot(
    index='loan_id',
    on=pl.col('^.*_payment$'),  # Regex pattern for columns
    variable_name='month',
    value_name='payment_amount'
)
```

### Real-World Example: Originator Data Restructuring
```python
# Originators often send data like this (wide format):
# loan_id | balance_2024_01 | balance_2024_02 | balance_2024_03
# 123     | 10000           | 9500            | 9000

# Need it like this for analysis (long format):
# loan_id | month    | balance
# 123     | 2024_01  | 10000
# 123     | 2024_02  | 9500
# 123     | 2024_03  | 9000

# Pandas
df_long = df.melt(
    id_vars=['loan_id'],
    value_vars=['balance_2024_01', 'balance_2024_02', 'balance_2024_03'],
    var_name='month',
    value_name='balance'
)
# Clean up month column
df_long['month'] = df_long['month'].str.replace('balance_', '')

# Polars
df_long = (
    df.unpivot(
        index='loan_id',
        on=[col for col in df.columns if col.startswith('balance_')],
        variable_name='month',
        value_name='balance'
    )
    .with_columns(
        pl.col('month').str.replace('balance_', '')
    )
)
```

### Melt Multiple Value Column Groups
```python
# Example: Data has both balance and payment columns for each month
# loan_id | balance_jan | payment_jan | balance_feb | payment_feb
# 123     | 10000       | 500         | 9500        | 500

# Pandas - need to melt twice and merge
balance_long = df.melt(
    id_vars=['loan_id'],
    value_vars=['balance_jan', 'balance_feb'],
    var_name='month',
    value_name='balance'
)
balance_long['month'] = balance_long['month'].str.replace('balance_', '')

payment_long = df.melt(
    id_vars=['loan_id'],
    value_vars=['payment_jan', 'payment_feb'],
    var_name='month',
    value_name='payment'
)
payment_long['month'] = payment_long['month'].str.replace('payment_', '')

df_long = pd.merge(balance_long, payment_long, on=['loan_id', 'month'])

# Polars - can unpivot multiple value columns at once
df_long = (
    df.unpivot(
        index='loan_id',
        on=['balance_jan', 'payment_jan', 'balance_feb', 'payment_feb'],
        variable_name='metric',
        value_name='value'
    )
    # Then pivot back to get balance and payment as separate columns
    .with_columns([
        pl.col('metric').str.extract(r'(balance|payment)', 1).alias('metric_type'),
        pl.col('metric').str.extract(r'_(jan|feb)', 1).alias('month')
    ])
    .pivot(
        values='value',
        index=['loan_id', 'month'],
        columns='metric_type'
    )
)
```

---

## Concatenating DataFrames

```python
# Pandas - Vertical (stacking rows)
pd.concat([df1, df2], ignore_index=True)

# Polars
pl.concat([df1, df2], how='vertical')

# Pandas - Horizontal (adding columns)
pd.concat([df1, df2], axis=1)

# Polars
pl.concat([df1, df2], how='horizontal')
```

---

## Dropping Columns

```python
# Pandas
df.drop(columns=['col1', 'col2'])

# Polars
df.drop(['col1', 'col2'])
```

---

## Dropping Duplicates

```python
# Pandas
df.drop_duplicates()
df.drop_duplicates(subset=['loan_id'])
df.drop_duplicates(subset=['loan_id'], keep='last')

# Polars
df.unique()
df.unique(subset=['loan_id'])
df.unique(subset=['loan_id'], keep='last')
```

---

## Writing Data

```python
# Pandas
df.to_csv('output.csv', index=False)
df.to_parquet('output.parquet')

# Polars
df.write_csv('output.csv')
df.write_parquet('output.parquet')
```

---

## Key Differences to Remember

1. **Immutability**: Polars operations return new DataFrames; you need to reassign or chain
   ```python
   # Pandas (can modify in place)
   df.drop(columns=['col1'], inplace=True)
   
   # Polars (always returns new DataFrame)
   df = df.drop(['col1'])
   ```

2. **Expression System**: Polars uses `pl.col()` for column references in expressions
   ```python
   # Pandas
   df[df['balance'] > 1000]
   
   # Polars
   df.filter(pl.col('balance') > 1000)
   ```

3. **Lazy Evaluation**: Use `scan_*` instead of `read_*` for lazy evaluation
   ```python
   df = pl.scan_parquet('large_file.parquet')  # Doesn't load yet
   df = df.filter(...)  # Still not loaded
   df = df.collect()  # Now it executes and loads
   ```

4. **Over vs GroupBy**: Window functions use `.over()` instead of `.groupby()`
   ```python
   # Pandas
   df['rank'] = df.groupby('originator')['balance'].rank()
   
   # Polars
   df = df.with_columns(
       pl.col('balance').rank().over('originator').alias('rank')
   )
   ```

---

## Performance Tips

1. **Use lazy evaluation for large datasets**:
   ```python
   df = (
       pl.scan_parquet('large_file.parquet')
       .filter(pl.col('balance') > 1000)
       .select(['loan_id', 'balance', 'originator'])
       .collect()
   )
   ```

2. **Polars automatically uses all CPU cores** - no need for explicit parallelization

3. **Use `.with_columns()` for multiple column operations** - more efficient than chaining

4. **Read only the columns you need**:
   ```python
   df = pl.read_parquet('loans.parquet', columns=['loan_id', 'balance'])
   ```

---

## Converting Between Pandas and Polars

```python
# Pandas to Polars
polars_df = pl.from_pandas(pandas_df)

# Polars to Pandas
pandas_df = polars_df.to_pandas()
```

---

## When to Use Which

**Use Pandas when:**
- Quick exploratory analysis
- Small datasets (<500MB)
- Need extensive third-party library integration
- Team is more familiar with pandas

**Use Polars when:**
- Processing multi-GB files
- Building production ETL pipelines
- Speed is critical
- Working with parquet files
- Need efficient memory usage
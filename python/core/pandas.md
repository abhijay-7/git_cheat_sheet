# Pandas In-Depth Cheatsheet

## Installation and Setup

```bash
# Install pandas
pip install pandas

# Install with additional packages
pip install pandas numpy matplotlib seaborn

# Import pandas
import pandas as pd
import numpy as np
```

## Core Data Structures

### Series (1D Data)
```python
# Creating Series
s = pd.Series([1, 2, 3, 4, 5])
s = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s = pd.Series({'a': 1, 'b': 2, 'c': 3})
s = pd.Series(np.random.randn(5))

# Series properties
print(s.values)     # Get values as numpy array
print(s.index)      # Get index
print(s.dtype)      # Data type
print(s.shape)      # Shape
print(s.size)       # Number of elements

# Series operations
s.head(3)           # First 3 elements
s.tail(3)           # Last 3 elements
s.describe()        # Summary statistics
s.value_counts()    # Count unique values
s.unique()          # Unique values
s.nunique()         # Number of unique values

# Series indexing and selection
s[0]                # By position
s['a']              # By label
s[[0, 2]]           # Multiple positions
s[['a', 'c']]       # Multiple labels
s[s > s.median()]   # Boolean indexing
```

### DataFrame (2D Data)
```python
# Creating DataFrames
df = pd.DataFrame([[1, 2], [3, 4]], columns=['A', 'B'])
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})
df = pd.DataFrame(np.random.randn(4, 3), columns=['A', 'B', 'C'])

# From dictionary of Series
df = pd.DataFrame({
    'A': pd.Series([1, 2, 3]),
    'B': pd.Series([4, 5, 6])
})

# DataFrame properties
print(df.shape)     # (rows, columns)
print(df.size)      # Total elements
print(df.ndim)      # Number of dimensions
print(df.dtypes)    # Data types of columns
print(df.columns)   # Column names
print(df.index)     # Row index
print(df.info())    # Comprehensive info
print(df.memory_usage())  # Memory usage

# DataFrame inspection
df.head(n=5)        # First n rows
df.tail(n=5)        # Last n rows
df.sample(n=3)      # Random n rows
df.describe()       # Summary statistics
df.describe(include='all')  # All columns
```

## Data Input/Output Operations

### Reading Data
```python
# CSV files
df = pd.read_csv('file.csv')
df = pd.read_csv('file.csv', 
                 sep=';',           # Custom separator
                 header=0,          # Header row
                 names=['A', 'B'],  # Column names
                 index_col=0,       # Index column
                 usecols=['A', 'B'], # Select columns
                 dtype={'A': int},   # Data types
                 parse_dates=['date'], # Parse dates
                 na_values=['NULL', 'N/A'], # Null values
                 skiprows=1,        # Skip rows
                 nrows=1000,        # Limit rows
                 encoding='utf-8')  # Encoding

# Excel files
df = pd.read_excel('file.xlsx')
df = pd.read_excel('file.xlsx', 
                   sheet_name='Sheet1',  # Specific sheet
                   header=[0, 1],        # Multi-level header
                   skiprows=2,           # Skip rows
                   usecols='A:C')        # Column range

# JSON files
df = pd.read_json('file.json')
df = pd.read_json('file.json', orient='records')  # Different orientations

# SQL databases
import sqlite3
conn = sqlite3.connect('database.db')
df = pd.read_sql('SELECT * FROM table', conn)
df = pd.read_sql_query('SELECT * FROM table WHERE col > 100', conn)

# HTML tables
df = pd.read_html('https://example.com/table.html')

# Clipboard
df = pd.read_clipboard()

# Other formats
df = pd.read_parquet('file.parquet')
df = pd.read_feather('file.feather')
df = pd.read_pickle('file.pkl')
```

### Writing Data
```python
# CSV files
df.to_csv('output.csv')
df.to_csv('output.csv',
          sep=';',           # Custom separator
          index=False,       # Don't write index
          header=True,       # Write header
          columns=['A', 'B'], # Select columns
          na_rep='NULL',     # Null representation
          encoding='utf-8')  # Encoding

# Excel files
df.to_excel('output.xlsx')
df.to_excel('output.xlsx',
            sheet_name='Data',  # Sheet name
            index=False,        # Don't write index
            startrow=1,         # Start row
            startcol=1)         # Start column

# Multiple sheets
with pd.ExcelWriter('output.xlsx') as writer:
    df1.to_excel(writer, sheet_name='Sheet1')
    df2.to_excel(writer, sheet_name='Sheet2')

# JSON files
df.to_json('output.json')
df.to_json('output.json', orient='records', indent=2)

# SQL databases
df.to_sql('table_name', conn, if_exists='replace', index=False)

# Other formats
df.to_parquet('output.parquet')
df.to_feather('output.feather')
df.to_pickle('output.pkl')
df.to_html('output.html')
```

## Data Selection and Indexing

### Column Selection
```python
# Single column
df['A']             # Returns Series
df[['A']]           # Returns DataFrame

# Multiple columns
df[['A', 'B']]
df.filter(['A', 'B'])
df.filter(regex='^A')  # Columns starting with 'A'

# Column operations
df.columns.tolist()  # Get column names as list
df.select_dtypes(include=[np.number])  # Select numeric columns
df.select_dtypes(exclude=[np.number])  # Exclude numeric columns
```

### Row Selection
```python
# By position (iloc)
df.iloc[0]          # First row
df.iloc[0:3]        # First 3 rows
df.iloc[-1]         # Last row
df.iloc[:, 0:2]     # All rows, first 2 columns
df.iloc[0:3, 0:2]   # First 3 rows, first 2 columns

# By label (loc)
df.loc[0]           # Row with index 0
df.loc[0:2]         # Rows with index 0 to 2
df.loc[:, 'A':'C']  # All rows, columns A to C
df.loc[0:2, 'A':'C']  # Rows 0-2, columns A-C

# Boolean indexing
df[df['A'] > 5]     # Rows where A > 5
df[df['A'].isin([1, 2, 3])]  # Rows where A is in list
df[df['A'].between(1, 5)]    # Rows where A is between 1 and 5
df[df['A'].str.contains('text')]  # String contains

# Complex conditions
df[(df['A'] > 5) & (df['B'] < 10)]   # AND condition
df[(df['A'] > 5) | (df['B'] < 10)]   # OR condition
df[~(df['A'] > 5)]                   # NOT condition

# Query method
df.query('A > 5')
df.query('A > 5 and B < 10')
df.query('A in [1, 2, 3]')
df.query('A > @threshold')  # Use variable
```

### Setting Values
```python
# Single value
df.loc[0, 'A'] = 100
df.iloc[0, 0] = 100

# Multiple values
df.loc[0:2, 'A'] = [1, 2, 3]
df.loc[df['A'] > 5, 'B'] = 0  # Conditional assignment

# New column
df['C'] = df['A'] + df['B']
df['D'] = np.where(df['A'] > 5, 'High', 'Low')  # Conditional

# Using assign
df = df.assign(E=df['A'] * 2, F=lambda x: x['A'] + x['B'])
```

## Index Operations

### Setting and Resetting Index
```python
# Set index
df.set_index('A')                    # Set column A as index
df.set_index('A', inplace=True)      # Modify in place
df.set_index(['A', 'B'])             # Multi-level index
df.set_index('A', drop=False)        # Keep column

# Reset index
df.reset_index()                     # Move index to column
df.reset_index(inplace=True)         # Modify in place
df.reset_index(drop=True)            # Drop index
df.reset_index(level=0)              # Reset specific level

# Index operations
df.reindex([0, 2, 4])               # Reorder by index
df.reindex(columns=['B', 'A'])      # Reorder columns
df.reindex_like(other_df)           # Match another DataFrame's index
```

### MultiIndex Operations
```python
# Creating MultiIndex
arrays = [['A', 'A', 'B', 'B'], [1, 2, 1, 2]]
index = pd.MultiIndex.from_arrays(arrays, names=['first', 'second'])
df = pd.DataFrame(np.random.randn(4, 2), index=index)

# From tuples
tuples = [('A', 1), ('A', 2), ('B', 1), ('B', 2)]
index = pd.MultiIndex.from_tuples(tuples, names=['first', 'second'])

# From product
index = pd.MultiIndex.from_product([['A', 'B'], [1, 2]], 
                                   names=['first', 'second'])

# Accessing MultiIndex
df.loc['A']                         # All rows with first level 'A'
df.loc[('A', 1)]                    # Specific combination
df.loc[:, ('A', 1)]                 # Column access
df.xs('A', level='first')           # Cross-section

# MultiIndex operations
df.swaplevel()                      # Swap index levels
df.stack()                          # Pivot columns to index
df.unstack()                        # Pivot index to columns
df.unstack(level=0)                 # Unstack specific level
```

## Data Cleaning and Preparation

### Handling Missing Data
```python
# Detecting missing values
df.isnull()         # Boolean mask of null values
df.notnull()        # Boolean mask of non-null values
df.isnull().sum()   # Count nulls per column
df.isnull().any()   # Any nulls per column
df.isnull().all()   # All nulls per column

# Removing missing values
df.dropna()                      # Drop rows with any null
df.dropna(axis=1)                # Drop columns with any null
df.dropna(subset=['A', 'B'])     # Drop rows with nulls in specific columns
df.dropna(thresh=2)              # Keep rows with at least 2 non-null values
df.dropna(how='all')             # Drop rows where all values are null

# Filling missing values
df.fillna(0)                     # Fill with constant
df.fillna({'A': 0, 'B': 1})      # Fill different values per column
df.fillna(method='ffill')        # Forward fill
df.fillna(method='bfill')        # Backward fill
df.fillna(df.mean())             # Fill with mean
df.fillna(df.median())           # Fill with median
df.fillna(df.mode().iloc[0])     # Fill with mode

# Interpolation
df.interpolate()                 # Linear interpolation
df.interpolate(method='polynomial', order=2)  # Polynomial
df.interpolate(method='time')    # Time-based interpolation
```

### Handling Duplicates
```python
# Detecting duplicates
df.duplicated()                  # Boolean mask of duplicates
df.duplicated(subset=['A'])      # Based on specific columns
df.duplicated(keep='first')      # Mark all but first as duplicate
df.duplicated(keep='last')       # Mark all but last as duplicate
df.duplicated(keep=False)        # Mark all duplicates

# Removing duplicates
df.drop_duplicates()             # Remove duplicate rows
df.drop_duplicates(subset=['A']) # Based on specific columns
df.drop_duplicates(keep='first') # Keep first occurrence
df.drop_duplicates(keep='last')  # Keep last occurrence
```

### Data Type Conversions
```python
# Convert data types
df['A'] = df['A'].astype(int)
df['B'] = df['B'].astype(float)
df['C'] = df['C'].astype(str)
df['D'] = df['D'].astype('category')

# Multiple columns
df = df.astype({'A': int, 'B': float, 'C': str})

# Numeric conversion with errors
pd.to_numeric(df['A'], errors='coerce')  # Invalid -> NaN
pd.to_numeric(df['A'], errors='ignore')  # Invalid -> original

# DateTime conversion
df['date'] = pd.to_datetime(df['date'])
df['date'] = pd.to_datetime(df['date'], format='%Y-%m-%d')
df['date'] = pd.to_datetime(df['date'], errors='coerce')

# Categorical data
df['category'] = df['category'].astype('category')
df['category'] = pd.Categorical(df['category'], 
                               categories=['low', 'medium', 'high'],
                               ordered=True)
```

## Data Transformation and Manipulation

### Applying Functions
```python
# Apply to columns (axis=0) or rows (axis=1)
df.apply(np.sum)                 # Sum each column
df.apply(np.sum, axis=1)         # Sum each row
df.apply(lambda x: x.max() - x.min())  # Range of each column

# Apply to specific columns
df[['A', 'B']].apply(np.sum)

# Applymap (element-wise)
df.applymap(lambda x: x**2)      # Square all elements
df.applymap(str.upper)           # Uppercase string elements

# Map (Series only)
df['A'].map({1: 'one', 2: 'two', 3: 'three'})
df['A'].map(lambda x: x**2)

# Transform (preserve shape)
df.transform(lambda x: x - x.mean())  # Center data
df.groupby('category').transform('mean')  # Group-wise mean
```

### String Operations
```python
# String methods (str accessor)
df['text'].str.len()             # String length
df['text'].str.upper()           # Uppercase
df['text'].str.lower()           # Lowercase
df['text'].str.title()           # Title case
df['text'].str.strip()           # Remove whitespace
df['text'].str.replace('old', 'new')  # Replace text
df['text'].str.contains('pattern')    # Contains pattern
df['text'].str.startswith('prefix')   # Starts with
df['text'].str.endswith('suffix')     # Ends with
df['text'].str.split(',')             # Split string
df['text'].str.split(',', expand=True)  # Split into columns
df['text'].str.extract(r'(\d+)')      # Extract pattern
df['text'].str.findall(r'\d+')        # Find all patterns
df['text'].str.slice(0, 3)            # Slice string
df['text'].str.pad(10, fillchar='0')  # Pad string
```

### DateTime Operations
```python
# DateTime properties
df['date'].dt.year               # Year
df['date'].dt.month              # Month
df['date'].dt.day                # Day
df['date'].dt.dayofweek          # Day of week (0=Monday)
df['date'].dt.dayname()          # Day name
df['date'].dt.quarter            # Quarter
df['date'].dt.is_leap_year       # Leap year
df['date'].dt.days_in_month      # Days in month

# DateTime operations
df['date'].dt.strftime('%Y-%m-%d')    # Format datetime
df['date'] + pd.Timedelta(days=30)    # Add time period
df['date'] - pd.DateOffset(months=1)  # Subtract time period

# Date ranges
pd.date_range('2023-01-01', periods=10, freq='D')    # Daily
pd.date_range('2023-01-01', periods=10, freq='W')    # Weekly
pd.date_range('2023-01-01', periods=10, freq='M')    # Monthly

# Resampling (time series)
df.set_index('date').resample('M').mean()  # Monthly average
df.set_index('date').resample('D').sum()   # Daily sum
```

## Grouping and Aggregation

### Basic Grouping
```python
# Group by single column
grouped = df.groupby('category')
grouped.sum()                    # Sum by group
grouped.mean()                   # Mean by group
grouped.count()                  # Count by group
grouped.size()                   # Size of each group
grouped.describe()               # Summary statistics by group

# Group by multiple columns
df.groupby(['category', 'subcategory']).sum()

# Group by function
df.groupby(df['date'].dt.month).mean()  # Group by month
df.groupby(len).sum()            # Group by string length
```

### Advanced Aggregation
```python
# Multiple aggregation functions
df.groupby('category').agg(['sum', 'mean', 'count'])

# Different functions for different columns
df.groupby('category').agg({
    'A': 'sum',
    'B': ['mean', 'std'],
    'C': 'count'
})

# Custom aggregation functions
df.groupby('category').agg({
    'A': lambda x: x.max() - x.min(),
    'B': 'sum'
})

# Named aggregation
df.groupby('category').agg(
    A_sum=('A', 'sum'),
    A_mean=('A', 'mean'),
    B_count=('B', 'count')
)

# Transform (preserve original shape)
df.groupby('category').transform('mean')  # Group-wise mean
df.groupby('category')['A'].transform(lambda x: x - x.mean())  # Center

# Filter groups
df.groupby('category').filter(lambda x: len(x) > 2)  # Groups with >2 rows
df.groupby('category').filter(lambda x: x['A'].sum() > 100)  # Custom filter
```

### Window Functions
```python
# Rolling windows
df['A'].rolling(window=3).mean()         # 3-period moving average
df['A'].rolling(window=3).sum()          # 3-period rolling sum
df['A'].rolling(window=3, min_periods=1).mean()  # Allow partial windows

# Expanding windows
df['A'].expanding().mean()               # Cumulative mean
df['A'].expanding().sum()                # Cumulative sum

# Exponential weighted windows
df['A'].ewm(span=3).mean()              # Exponential moving average
df['A'].ewm(alpha=0.1).mean()           # With decay parameter
```

## Reshaping and Pivot Operations

### Pivot Tables
```python
# Basic pivot table
df.pivot_table(values='sales', 
               index='product', 
               columns='month', 
               aggfunc='sum')

# Multiple values
df.pivot_table(values=['sales', 'profit'],
               index='product',
               columns='month',
               aggfunc='sum')

# Multiple aggregation functions
df.pivot_table(values='sales',
               index='product',
               columns='month',
               aggfunc=['sum', 'mean', 'count'])

# With margins (totals)
df.pivot_table(values='sales',
               index='product',
               columns='month',
               aggfunc='sum',
               margins=True,
               margins_name='Total')

# Fill missing values
df.pivot_table(values='sales',
               index='product',
               columns='month',
               aggfunc='sum',
               fill_value=0)
```

### Melting and Unpivoting
```python
# Melt (wide to long)
pd.melt(df, 
        id_vars=['product'],      # Identifier columns
        value_vars=['Jan', 'Feb'], # Columns to melt
        var_name='month',         # Variable column name
        value_name='sales')       # Value column name

# Melt all columns except id_vars
pd.melt(df, id_vars=['product'])

# Stack and unstack
df.set_index(['product', 'month']).stack()    # Stack columns to index
df.set_index(['product', 'month']).unstack()  # Unstack index to columns
df.unstack(level=0)                           # Unstack specific level
df.unstack(fill_value=0)                      # Fill missing values
```

### Concatenation and Merging
```python
# Concatenation
pd.concat([df1, df2])                    # Vertical concatenation
pd.concat([df1, df2], axis=1)            # Horizontal concatenation
pd.concat([df1, df2], ignore_index=True) # Reset index
pd.concat([df1, df2], keys=['A', 'B'])   # Add keys

# Merge (SQL-style joins)
# Inner join
pd.merge(df1, df2, on='key')
pd.merge(df1, df2, left_on='key1', right_on='key2')

# Left join
pd.merge(df1, df2, on='key', how='left')

# Right join
pd.merge(df1, df2, on='key', how='right')

# Outer join
pd.merge(df1, df2, on='key', how='outer')

# Multiple keys
pd.merge(df1, df2, on=['key1', 'key2'])

# Suffix for duplicate columns
pd.merge(df1, df2, on='key', suffixes=('_left', '_right'))

# Join (index-based merge)
df1.join(df2, how='left', lsuffix='_left', rsuffix='_right')
```

## Advanced Operations

### Categorical Data
```python
# Create categorical
df['grade'] = pd.Categorical(df['grade'], 
                            categories=['A', 'B', 'C', 'D', 'F'],
                            ordered=True)

# Categorical operations
df['grade'].cat.categories           # Get categories
df['grade'].cat.codes               # Get numeric codes
df['grade'].cat.add_categories(['A+'])  # Add category
df['grade'].cat.remove_categories(['F'])  # Remove category
df['grade'].cat.rename_categories({'A': 'Excellent'})  # Rename
df['grade'].cat.reorder_categories(['F', 'D', 'C', 'B', 'A'])  # Reorder

# Categorical aggregation
df.groupby('grade').mean()          # Respects category order
```

### Cross Tabulation
```python
# Cross tabulation
pd.crosstab(df['A'], df['B'])               # Frequency table
pd.crosstab(df['A'], df['B'], normalize=True)  # Proportions
pd.crosstab(df['A'], df['B'], margins=True)    # With margins
pd.crosstab([df['A'], df['C']], df['B'])       # Multiple rows
```

### Binning and Discretization
```python
# Cut (equal-width bins)
pd.cut(df['age'], bins=5)                    # 5 equal-width bins
pd.cut(df['age'], bins=[0, 18, 65, 100])     # Custom bin edges
pd.cut(df['age'], bins=5, labels=['A', 'B', 'C', 'D', 'E'])  # Custom labels

# Qcut (equal-frequency bins)
pd.qcut(df['score'], q=4)                    # Quartiles
pd.qcut(df['score'], q=10)                   # Deciles
pd.qcut(df['score'], q=[0, 0.25, 0.5, 0.75, 1.0])  # Custom quantiles
```

## Performance Optimization

### Memory Usage
```python
# Check memory usage
df.info(memory_usage='deep')
df.memory_usage(deep=True)

# Optimize data types
df['int_col'] = df['int_col'].astype('int32')      # Smaller int
df['float_col'] = df['float_col'].astype('float32') # Smaller float
df['str_col'] = df['str_col'].astype('category')    # Categorical for strings

# Downcast numeric types
df['int_col'] = pd.to_numeric(df['int_col'], downcast='integer')
df['float_col'] = pd.to_numeric(df['float_col'], downcast='float')
```

### Efficient Operations
```python
# Use vectorized operations instead of loops
# Bad
result = []
for i in range(len(df)):
    result.append(df.iloc[i]['A'] * 2)

# Good
result = df['A'] * 2

# Use query() for complex filtering
df.query('A > 5 and B < 10')  # Faster than boolean indexing

# Use eval() for complex expressions
df.eval('C = A + B')          # Faster than df['C'] = df['A'] + df['B']

# Chunking for large files
chunk_size = 10000
for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    process_chunk(chunk)
```

## Styling and Visualization

### DataFrame Styling
```python
# Basic styling
df.style.highlight_max()                    # Highlight maximum values
df.style.highlight_min()                    # Highlight minimum values
df.style.highlight_null()                   # Highlight null values

# Custom styling
df.style.applymap(lambda x: 'color: red' if x < 0 else 'color: black')
df.style.apply(lambda x: ['background: yellow' if v > x.mean() else '' for v in x])

# Formatting
df.style.format({'A': '{:.2f}', 'B': '{:.0%}'})  # Number formatting
df.style.format(precision=2)                      # Global precision

# Conditional formatting
df.style.background_gradient(cmap='viridis')      # Color gradient
df.style.bar(subset=['A', 'B'])                   # Data bars
```

### Quick Plotting
```python
# Line plot
df.plot()
df.plot(x='date', y='value')

# Different plot types
df.plot.bar()           # Bar plot
df.plot.barh()          # Horizontal bar plot
df.plot.hist()          # Histogram
df.plot.box()           # Box plot
df.plot.scatter(x='A', y='B')  # Scatter plot
df.plot.area()          # Area plot
df.plot.pie(y='values') # Pie chart

# Subplots
df.plot(subplots=True, layout=(2, 2), figsize=(10, 8))
```

## Best Practices and Common Patterns

### Method Chaining
```python
# Method chaining for readable code
result = (df
    .query('A > 5')
    .groupby('category')
    .agg({'B': 'sum', 'C': 'mean'})
    .reset_index()
    .sort_values('B', ascending=False)
    .head(10)
)
```

### Error Handling
```python
# Handle errors gracefully
try:
    df = pd.read_csv('file.csv')
except FileNotFoundError:
    print("File not found")
except pd.errors.EmptyDataError:
    print("File is empty")
except pd.errors.ParserError:
    print("Error parsing file")

# Validate data
assert df.shape[0] > 0, "DataFrame is empty"
assert 'required_column' in df.columns, "Required column missing"
assert df['numeric_column'].dtype in ['int64', 'float64'], "Column must be numeric"
```

### Common Workflows
```python
# Data cleaning pipeline
def clean_data(df):
    return (df
        .drop_duplicates()
        .dropna(subset=['important_column'])
        .assign(
            clean_text=lambda x: x['text_column'].str.strip().str.lower(),
            date_parsed=lambda x: pd.to_datetime(x['date_column'], errors='coerce')
        )
        .query('date_parsed >= "2020-01-01"')
        .reset_index(drop=True)
    )

# Aggregation pipeline  
def summarize_data(df):
    return (df
        .groupby(['category', 'subcategory'])
        .agg({
            'sales': ['sum', 'mean', 'count'],
            'profit': 'sum',
            'date': ['min', 'max']
        })
        .round(2)
        .reset_index()
    )
```
## 2.2 Data Manipulation and Analysis

Collecting data is only the first step; the real value in OSINT comes from analyzing that data to uncover insights, patterns, and connections. Raw data is often messy, incomplete, or in formats unsuitable for direct analysis. This section focuses on using two fundamental Python libraries, `numpy` and `pandas`, within Jupyter Notebook to clean, transform, manipulate, and analyze the data you've collected.

### `numpy` Essentials for Numerical Data

`numpy` (Numerical Python) is the foundational package for numerical computing in Python. While `pandas` (which we'll cover next) is built on top of `numpy`, understanding basic `numpy` concepts is helpful, especially when dealing with numerical arrays or performing mathematical operations efficiently.

Installation (usually included with Anaconda):

```python
# !pip install numpy
```

Key Concepts:

*   **`ndarray`:** The core object is the N-dimensional array. It's a grid of values, all of the same type, indexed by a tuple of non-negative integers. `numpy` arrays offer significant performance advantages over standard Python lists for numerical operations.
*   **Vectorized Operations:** `numpy` allows you to perform operations on entire arrays without writing explicit loops, which is much faster.
*   **Mathematical Functions:** Provides a vast library of mathematical functions that operate element-wise on arrays.

Example Usage:

```python
import numpy as np

# Create arrays
arr1 = np.array([1, 2, 3, 4, 5])
arr2 = np.array([[1.1, 2.2, 3.3], [4.4, 5.5, 6.6]]) # 2D array

print(f"Array 1: {arr1}")
print(f"Shape of Array 2: {arr2.shape}") # (rows, columns)
print(f"Data type of Array 2: {arr2.dtype}")

# Basic arithmetic (vectorized)
print(f"\narr1 + 10: {arr1 + 10}")
print(f"arr1 * 2: {arr1 * 2}")
print(f"arr1 + arr1: {arr1 + arr1}")

# Mathematical functions
print(f"\nSquare root of arr1: {np.sqrt(arr1)}")
print(f"Sine of arr1: {np.sin(arr1)}")

# Indexing and Slicing (similar to lists, but can be multi-dimensional)
print(f"\nElement at index 2 in arr1: {arr1[2]}")
print(f"Elements from index 1 to 3 in arr1: {arr1[1:4]}")
print(f"Element at row 0, column 1 in arr2: {arr2[0, 1]}") # Use comma for dimensions
print(f"First row of arr2: {arr2[0, :]}")
print(f"Second column of arr2: {arr2[:, 1]}")

# Boolean indexing
bool_idx = arr1 > 3
print(f"\nElements in arr1 greater than 3: {arr1[bool_idx]}")

# Basic statistics
print(f"\nMean of arr2: {np.mean(arr2)}")
print(f"Sum of arr1: {np.sum(arr1)}")
print(f"Max value in arr2: {np.max(arr2)}")
```

While we won't delve deeply into complex `numpy` operations in this OSINT course, recognizing its role and basic array manipulations is beneficial, especially when preparing numerical data for `pandas` or visualization libraries.

### `pandas`: The Powerhouse for Data Analysis

`pandas` is the essential library for data manipulation and analysis in Python. It provides high-performance, easy-to-use data structures and data analysis tools, making it indispensable for OSINT work involving structured or semi-structured data.

Installation (usually included with Anaconda):

```python
# !pip install pandas
```

**Core Data Structures**

*   **`Series`:** A one-dimensional labeled array capable of holding any data type (integers, strings, floating-point numbers, Python objects, etc.). It's like a column in a spreadsheet or a `numpy` array with an associated index.
*   **`DataFrame`:** A two-dimensional, size-mutable, potentially heterogeneous tabular data structure with labeled axes (rows and columns). It's the primary `pandas` object, analogous to a spreadsheet, SQL table, or a dictionary of `Series` objects.

```python
import pandas as pd

# Create a Series
s = pd.Series([10, 20, 30, np.nan, 50], index=["a", "b", "c", "d", "e"], name="MyNumbers")
print("Pandas Series:")
print(s)

# Create a DataFrame from a dictionary
data = {
    "URL": ["site-a.com", "site-b.org", "site-c.net", "site-a.com"],
    "Type": ["Blog", "Forum", "News", "Blog"],
    "Visits": [150, 80, 220, 165],
    "Date": pd.to_datetime(["2025-04-28", "2025-04-28", "2025-04-27", "2025-04-29"])
}
df = pd.DataFrame(data)
print("\nPandas DataFrame:")
print(df)
```

**Data Loading and Saving**

`pandas` excels at reading data from various formats and writing results back.

```python
# Assuming scraped_data is a list of dictionaries from Section 2.1
# scraped_data = [
#     {"text": "Quote 1...", "author": "Author A", "tags": ["tag1", "tag2"]},
#     {"text": "Quote 2...", "author": "Author B", "tags": ["tag3"]}
# ]
# df_scraped = pd.DataFrame(scraped_data)

# --- Writing Data ---
# Save to CSV
# df_scraped.to_csv("scraped_quotes.csv", index=False) # index=False prevents writing the DataFrame index
# print("Saved scraped data to scraped_quotes.csv")

# Save to JSON (different orientations possible)
# df_scraped.to_json("scraped_quotes.json", orient="records", indent=4)
# print("Saved scraped data to scraped_quotes.json")

# Save to Excel (requires openpyxl or xlsxwriter)
# !pip install openpyxl
# df_scraped.to_excel("scraped_quotes.xlsx", index=False, sheet_name="Quotes")
# print("Saved scraped data to scraped_quotes.xlsx")

# --- Reading Data ---
try:
    # Read from CSV
    df_from_csv = pd.read_csv("findings.csv") # Assuming findings.csv exists from Module 1
    print("\nDataFrame read from findings.csv:")
    print(df_from_csv.head()) # Display first 5 rows
except FileNotFoundError:
    print("\nfindings.csv not found. Skipping CSV read example.")

# Read from JSON
# try:
#     df_from_json = pd.read_json("scraped_quotes.json", orient="records")
#     print("\nDataFrame read from scraped_quotes.json:")
#     print(df_from_json.head())
# except FileNotFoundError:
#     print("\nscraped_quotes.json not found. Skipping JSON read example.")

# Read from Excel
# try:
#     df_from_excel = pd.read_excel("scraped_quotes.xlsx", sheet_name="Quotes")
#     print("\nDataFrame read from scraped_quotes.xlsx:")
#     print(df_from_excel.head())
# except FileNotFoundError:
#     print("\nscraped_quotes.xlsx not found. Skipping Excel read example.")

# Connecting to Databases (Conceptual - requires specific libraries like sqlalchemy, psycopg2, etc.)
# from sqlalchemy import create_engine
# engine = create_engine("postgresql://user:password@host:port/database")
# df_from_db = pd.read_sql("SELECT * FROM my_osint_table", engine)
```

**Data Inspection**

Understanding your data is the first step in analysis.

```python
print("\n--- Data Inspection ---")
print("DataFrame Info:")
df.info() # Concise summary: index dtype, column dtypes, non-null values, memory usage

print("\nDataFrame Description (numerical columns):")
print(df.describe()) # Summary statistics for numerical columns

print("\nDataFrame Description (all columns):")
print(df.describe(include=\"all\")) # Include stats for non-numerical columns

print("\nFirst 3 rows:")
print(df.head(3))

print("\nLast 2 rows:")
print(df.tail(2))

print("\nColumn Names:")
print(df.columns)

print("\nIndex:")
print(df.index)

print("\nValue Counts for 'Type' column:")
print(df["Type"].value_counts())
```

**Selection and Indexing**

Accessing specific parts of your data.

```python
print("\n--- Selection and Indexing ---")
# Select a single column (returns a Series)
print("URL Column:")
print(df["URL"])

# Select multiple columns (returns a DataFrame)
print("\nURL and Visits Columns:")
print(df[["URL", "Visits"]])

# Select rows by label (index) using .loc
# Let's set URL as index first for meaningful label-based access
df_indexed = df.set_index("URL") 
# print("\nDataFrame with URL as index:")
# print(df_indexed)
# print("\nRow with index 'site-b.org':")
# print(df_indexed.loc["site-b.org"]) # Select single row by label
# print("\nRows 'site-a.com':") # Select multiple rows with the same label
# print(df_indexed.loc["site-a.com"]) 

# Select rows by position using .iloc
print("\nRow at position 0:")
print(df.iloc[0])
print("\nRows at positions 1 and 2:")
print(df.iloc[1:3]) # Slices like Python lists
print("\nRows at positions 0 and 3:")
print(df.iloc[[0, 3]]) # Use list for specific positions

# Select specific rows and columns
print("\nVisits for rows 0 and 2:")
print(df.loc[[0, 2], "Visits"]) # Using .loc with default integer index
print("\nType and Date for row 1:")
print(df.iloc[1, [1, 3]]) # Using .iloc

# Boolean Indexing (very powerful for filtering)
print("\nRows where Visits > 100:")
print(df[df["Visits"] > 100])

print("\nRows where Type is 'Blog':")
print(df[df["Type"] == "Blog"])

print("\nRows where Type is 'Forum' AND Visits < 100:")
print(df[(df["Type"] == "Forum") & (df["Visits"] < 100)])

print("\nRows where URL contains 'site-a':")
print(df[df["URL"].str.contains("site-a")])
```

**Data Cleaning**

Real-world OSINT data is rarely perfect. Cleaning is essential.

```python
# Add some missing data for demonstration
df.loc[1, "Visits"] = np.nan
df.loc[3, "Type"] = None
print("\n--- Data Cleaning ---")
print("DataFrame with missing values:")
print(df)

# Check for missing values
print("\nMissing values per column:")
print(df.isnull().sum())

# Drop rows with any missing values
df_dropped_na = df.dropna()
print("\nDataFrame after dropping rows with NA:")
print(df_dropped_na)

# Fill missing values
# Fill Visits with the mean
mean_visits = df["Visits"].mean()
df_filled = df.copy() # Work on a copy
df_filled["Visits"].fillna(mean_visits, inplace=True)
# Fill Type with a specific value
df_filled["Type"].fillna("Unknown", inplace=True)
print("\nDataFrame after filling NA values:")
print(df_filled)

# Check for duplicates
print(f"\nNumber of duplicate rows: {df.duplicated().sum()}")
print("Duplicate rows:")
print(df[df.duplicated(keep=False)]) # Show all occurrences of duplicates

# Remove duplicate rows (keeping the first occurrence)
df_no_duplicates = df.drop_duplicates()
print("\nDataFrame after removing duplicates (keeping first):")
print(df_no_duplicates)

# Data Type Conversion
# Example: Convert Visits to integer after filling NA
df_filled["Visits"] = df_filled["Visits"].astype(int)
print("\nDataFrame with Visits converted to int:")
df_filled.info()
```

**Data Transformation**

Modifying data to suit your analysis needs.

```python
print("\n--- Data Transformation ---")
# Apply a function to a column
def categorize_visits(visits):
    if visits > 200:
        return "High"
    elif visits > 100:
        return "Medium"
    else:
        return "Low"

df["VisitCategory"] = df["Visits"].apply(categorize_visits)
print("\nDataFrame with Visit Category:")
print(df)

# Using .map for value replacement
type_mapping = {"Blog": "Content Site", "Forum": "Community Site", "News": "Content Site"}
df["SiteCategory"] = df["Type"].map(type_mapping).fillna("Other")
print("\nDataFrame with Site Category (using map):")
print(df)

# Renaming columns
df_renamed = df.rename(columns={"Visits": "PageViews", "Date": "RecordDate"})
print("\nDataFrame with renamed columns:")
print(df_renamed.head())

# Extracting information from strings (e.g., domain from URL)
df["Domain"] = df["URL"].str.split(".").str[0]
print("\nDataFrame with extracted Domain:")
print(df)
```

**Grouping and Aggregation**

Summarizing data based on categories.

```python
print("\n--- Grouping and Aggregation ---")
# Group by 'Type' and calculate mean visits
print("Mean visits per Type:")
print(df.groupby("Type")["Visits"].mean())

# Group by 'Type' and calculate multiple aggregations
print("\nAggregations per Type:")
print(df.groupby("Type")["Visits"].agg(["mean", "sum", "count", "min", "max"]))

# Group by multiple columns
print("\nSum of visits per Type and Domain:")
print(df.groupby(["Type", "Domain"])["Visits"].sum())

# Get size of each group
print("\nCount of entries per Type:")
print(df.groupby("Type").size())
```

**Merging and Joining DataFrames**

Combining data from different sources is common in OSINT.

```python
# Create another DataFrame for merging
data2 = {
    "URL": ["site-a.com", "site-b.org", "site-d.edu"],
    "Registrar": ["RegA", "RegB", "RegC"],
    "Country": ["USA", "UK", "USA"]
}
df_registrar = pd.DataFrame(data2)

print("\n--- Merging and Joining ---")
print("Original DataFrame:")
print(df)
print("\nRegistrar DataFrame:")
print(df_registrar)

# Merge based on the 'URL' column (inner join by default)
df_merged = pd.merge(df, df_registrar, on="URL", how="inner")
print("\nInner Merge Result:")
print(df_merged)

# Left join (keep all rows from df, match from df_registrar)
df_left_merged = pd.merge(df, df_registrar, on="URL", how="left")
print("\nLeft Merge Result:")
print(df_left_merged)

# Concatenating DataFrames (stacking rows)
df_concat = pd.concat([df.head(2), df.tail(1)], ignore_index=True)
print("\nConcatenated DataFrame (rows):")
print(df_concat)
```

Mastering `pandas` allows you to efficiently handle the diverse and often messy data encountered in OSINT investigations, preparing it for meaningful analysis and visualization.

### Assignment 3: Data Cleaning and Exploratory Analysis

**Objective:** Apply `pandas` skills to clean and perform exploratory data analysis (EDA) on a dataset.

**Instructions:**
1.  Use the data you collected in Assignment 2 (either the scraped data or the API data, or combine them if meaningful). Alternatively, use a provided OSINT-related dataset (e.g., a sample list of domains with registration info, social media posts, breach data - ensure ethical sourcing).
2.  Load the data into a `pandas` DataFrame.
3.  **Inspect the Data:** Use `.info()`, `.describe()`, `.head()`, `.tail()`, `.value_counts()` to understand the structure, data types, and initial content.
4.  **Clean the Data:**
    *   Identify and handle missing values (e.g., drop rows/columns, fill with appropriate values like mean, median, mode, or a specific string like "Unknown"). Justify your choices.
    *   Identify and handle duplicate entries.
    *   Correct any obvious errors or inconsistencies (e.g., inconsistent capitalization, incorrect data types). Convert columns to appropriate types (e.g., numbers, dates).
5.  **Transform and Enrich (Optional but Recommended):**
    *   Create new columns based on existing data (e.g., extract domain names from URLs, calculate time differences, categorize numerical values).
    *   Rename columns for clarity.
6.  **Perform Exploratory Analysis:**
    *   Use grouping (`.groupby()`) and aggregation (`.agg()`, `.mean()`, `.sum()`, `.count()`, etc.) to summarize the data based on relevant categories.
    *   Calculate basic statistics.
    *   Answer at least 3 specific questions about the data using your analysis (e.g., "What are the most common types of sources?", "What is the average frequency of posts per user?", "Which domains have the oldest registration dates?").
7.  **Document:** Clearly document each step of your cleaning and analysis process in the Jupyter Notebook using Markdown cells. Explain your reasoning for cleaning decisions and interpret the results of your analysis.

**Deliverables:**
- A Jupyter Notebook (.ipynb file) containing your code, detailed explanations, and analysis results.
- The cleaned dataset (optional, if significantly different from the original).

**Evaluation Criteria:**
- Correct loading and inspection of data.
- Appropriate identification and handling of missing values and duplicates.
- Successful data type conversions and transformations.
- Meaningful exploratory analysis using grouping and aggregation.
- Clear answers to the defined questions based on the analysis.
- Quality of documentation and explanation within the notebook.

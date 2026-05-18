# 📘 Assignment 2: Mastering Big Data Handling - Airbnb Listings Analysis

## 📌 Task 1: Dataset Selection
In this assignment, we tackle the challenges of handling massive datasets using Python. The chosen dataset must exceed 700MB to properly demonstrate Big Data handling techniques.

- **Dataset Name:** Airbnb Listings Dataset
- **Source:** Inside Airbnb / Open Data portals
- **Domain:** Hospitality / Property Rental
- **File Size:** ~1.85 GB (1846.24 MB)
- **Dimensions:** 494,954 rows × 89 columns

This dataset is rich enough for comprehensive exploratory analysis and provides a realistic scenario where traditional in-memory Pandas processing struggles with memory errors or slow execution times.

---

## 📌 Task 2: Load and Inspect Data
To avoid memory overflow right at the start, we first inspect the dataset by loading just the top 5 rows and extracting the column headers.

### Code Snippet:
```python
import os
import pandas as pd

file_path = "/content/drive/MyDrive/HPDP_A2/airbnb_listings.csv"

# Check file size
print("File size:", round(os.path.getsize(file_path) / (1024**2), 2), "MB")

# Load only 5 rows to understand the structure
preview = pd.read_csv(file_path, sep=';', nrows=5)
print("Preview shape:", preview.shape)
```
### Result / Output:
- **File Size:** 1846.24 MB
- **Preview Shape:** (5, 89)
This initial inspection allowed us to identify the 89 columns and plan our optimization strategies without crashing the Colab runtime.

---

## 📌 Task 3: Apply Big Data Handling Strategies

### Strategy 1: Load Less Data
Instead of loading all 89 columns, we identified **20 essential columns** (e.g., Price, Room Type, Location, Review Scores) necessary for our analysis and loaded only those using the `usecols` parameter.

**Code Snippet:**
```python
selected_cols = ["ID", "Name", "City", "Country", "Room Type", "Price", "Number of Reviews", ...] # 20 columns

import time
start = time.time()
df_less = pd.read_csv(working_file, sep=';', usecols=selected_cols, nrows=100000)
end = time.time()

print("Shape:", df_less.shape)
print("Loading time:", round(end - start, 2), "seconds")
print("Memory usage:", round(df_less.memory_usage(deep=True).sum() / (1024**2), 2), "MB")
```
**Results:**
- **Original Memory (100k rows, 89 cols):** 649.52 MB
- **Optimized Memory (100k rows, 20 cols):** 138.69 MB (**78.6% Reduction**)
- **Loading Time Drop:** From 9.85s to 4.81s.

### Strategy 2: Use Chunking
To process the entire dataset without exhausting RAM, we read the data in manageable batches (`chunksize=100000`), calculating aggregated statistics continuously.

**Code Snippet:**
```python
chunk_size = 100000
room_review_sum = {}
room_count = {}

for chunk in pd.read_csv(working_file, sep=';', usecols=["Room Type", "Number of Reviews"], chunksize=chunk_size):
    chunk["Number of Reviews"] = pd.to_numeric(chunk["Number of Reviews"], errors="coerce")
    grouped = chunk.groupby("Room Type")["Number of Reviews"].agg(["sum", "count"])
    for room_type, row in grouped.iterrows():
        room_review_sum[room_type] = room_review_sum.get(room_type, 0) + row["sum"]
        room_count[room_type] = room_count.get(room_type, 0) + row["count"]

chunk_result = {rt: room_review_sum[rt] / room_count[rt] for rt in room_review_sum}
```
**Results:**
- **Execution Time:** ~25.82 seconds to complete over the full 494k rows.
- Eliminates out-of-memory issues completely by streaming data from disk.

### Strategy 3: Optimize Data Types
By default, Pandas assigns memory-heavy types (`float64`, `object`). We converted string columns with low cardinality to `category`, and downcasted numeric types to `float32` and `int32`.

**Code Snippet:**
```python
# Convert to category
category_cols = ["City", "Country", "Property Type", "Room Type"]
for col in category_cols:
    df_opt[col] = df_opt[col].astype("category")

# Downcast integer and floats
df_opt["Number of Reviews"] = pd.to_numeric(df_opt["Number of Reviews"], downcast="integer")
df_opt["Price"] = pd.to_numeric(df_opt["Price"], downcast="float")
```
**Results:**
- **Further Memory Reduction:** Reduced the 20-column DataFrame memory usage from **138.69 MB to 110.82 MB (20.09% additional reduction)**.

### Strategy 4: Sampling
To enable rapid prototyping of data pipelines and visual EDA, we load a subset of the dataset and perform a 10% random sample.

**Code Snippet:**
```python
df_sample_base = pd.read_csv(working_file, sep=';', usecols=selected_cols, nrows=300000)
# Extract 10% sample
sample_df = df_sample_base.sample(frac=0.1, random_state=42)
print("10% sampled shape:", sample_df.shape)
```
**Results:**
- **Sample Size:** 30,000 rows.
- **Benefit:** Super fast development cycles. Final pipelines can then be applied to the full dataset.

### Strategy 5: Parallel Processing with Dask (and Polars)
We utilized **Dask** to read and process the large file in parallel, and additionally tested **Polars** (lazy execution) to compare modern framework efficiencies.

**Code Snippet (Dask):**
```python
import dask.dataframe as dd
ddf = dd.read_csv(working_file, sep=';', usecols=["Room Type", "Number of Reviews"], assume_missing=True)
dask_result = ddf.groupby("Room Type")["Number of Reviews"].mean().compute()
```
**Results:**
- **Dask Execution Time:** 33.84s on the full dataset using multi-core processing.
- **Polars Execution Time:** 6.09s (Fastest) utilizing Rust-based lazy querying.

---

## 📌 Task 4: Comparative Analysis

| Strategy | Memory Profile | Execution Time (Full Dataset) | Ease of Processing | Best Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Traditional Pandas** | Extremely High | Memory Error / ~48s | Easy | Small to Medium datasets |
| **Pandas (Selected Cols)** | Low (138MB/100k) | ~26.95s | Easy | Data with unused features |
| **Chunking** | Minimal (Bounded) | 25.82s | Moderate | Files exceeding max RAM |
| **Type Optimization** | Optimized (110MB) | N/A | Moderate | Production pipelines |
| **Dask (Parallel)** | Partitioned | 33.84s | Easy | Multi-core clusters |
| **Polars (Lazy)** | Highly Efficient | **6.09s** | Moderate | High-performance local |

**Observations on Comparisons:**
- While base Pandas is easy to write, trying to load the full 1.8GB file directly into memory risks crashing Colab.
- Memory usage was easily mitigated primarily by dropping unused columns (78% drop) and fine-tuning types (addl. 20% drop).
- In terms of pure execution time on a local instance, **Polars** heavily outperformed Dask's parallelization overhead.

---

## 📌 Task 5: Conclusion & Reflection

### Summary of Key Observations
Processing a nearly 2GB file taught us that scaling hardware (adding RAM) is not the only—or even the best—solution to Big Data problems. Smart software strategies drastically mitigate hardware constraints.

### Benefits and Limitations
- **Loading Less / Downcasting:** Very standard, robust practice. *Limitation:* Does not solve the fundamental single-thread speed bottleneck of Pandas.
- **Chunking:** Safest memory method for single machines. *Limitation:* Inherently slower as it continuously accesses the disk piecemeal, and makes logic operations (like GroupBys) much more complex to program.
- **Dask:** Great syntax compatibility with Pandas, scaling easily to clusters. *Limitation:* Overkill or slower for data that actually fits in local RAM (due to overhead).
- **Polars:** Blazing fast. *Limitation:* Has its own syntax syntax that must be learned, different from standard Pandas.

### Reflection
This assignment provided invaluable hands-on experience bridging the gap between theoretical Python coding and practical data engineering. We learned to prioritize *efficiency*. Applying a combination of optimizing data properties (filtering columns/types) alongside modern parallel processing engines allowed us to successfully query real-world enterprise-scale data locally.

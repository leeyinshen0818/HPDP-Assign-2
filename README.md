# 📘 Assignment 2: Mastering Big Data Handling

**Course:** High-Performance Data Processing (HPDP)  
**Dataset:** Airbnb Listings Analysis (~1.85 GB)

<a href="https://github.com/drshahizan/HPDP/stargazers"><img src="https://img.shields.io/github/stars/drshahizan/HPDP" alt="Stars Badge"/></a>

## 👥 Group Information

_(Please replace with your actual names and matric numbers prior to final submission)_

- **Student 1:** [Brendan Chia Yan Fei] ([A23CS0211])
- **Student 2:** [Name] ([Matric Number])

## 📌 Introduction

In this assignment, we address the challenge of managing and extracting insights from massive datasets that exceed standard memory capabilities. We utilized a **1.85 GB** file containing nearly 500,000 Airbnb listings to compare traditional, memory-intensive data analysis methods against optimized Big Data strategies.

## 📁 Repository Structure

To comply with the assignment structure, the repository is organized as follows:

- 📄 **[big_data.md](big_data.md)** — The comprehensive main Markdown report detailing out analysis, methodologies, implementation code snippets, and reflections across all 5 Tasks.
- 📄 **[README.md](README.md)** — This introductory brief and navigation page.
- 📓 **[big_data.ipynb](big_data.ipynb)** — The executed Google Colab / Jupyter Notebook containing all the functional Python code.

## 🚀 Learning Outcomes & Strategies Explored

Through the completion of this assignment, we successfully:

1. **Loaded Less Data:** Avoided parsing overhead by truncating 89 columns down to 20 essential features.
2. **Chunked Data:** Streamed the dataset from disk iteratively mapping and reducing 100,000 rows at a time to dodge OOM errors.
3. **Optimized Data Types:** Downcasted formats (e.g., `float64` to `float32`) and converted string variables to categories.
4. **Performed Sampling:** Generated rapid prototype pipelines using 10% statistical subsets.
5. **Executed Parallel processing:** Compared Pandas to distributed multi-core parallelization (**Dask**) and rust-based lazy evaluation (**Polars**).

## 🏆 Summary Result (Comparative Analysis)

Our comparative analysis found that the traditional approach of loading massive files directly into Pandas is highly inefficient.
By utilizing **Polars**, we achieved full-dataset aggregations in just **~6.09 seconds**, significantly outperforming traditional methods while maintaining a fractional memory footprint.

> 👉 **[Click here to read the full technical implementation and conclusions in `big_data.md`](big_data.md)**!

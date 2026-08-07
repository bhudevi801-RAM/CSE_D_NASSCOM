# Pandas ↔ Math Concepts

**Audience:** Absolute beginners (builds directly on the NumPy module — read that first)
**Goal:** Understand *why* Pandas is built the way it is by connecting each feature to the math idea behind it.

---

## 1. Why Pandas exists

NumPy gives us fast vectors and matrices, but a raw matrix has no memory of *what each row or column means*. Real-world data isn't just numbers — it's numbers **labeled** with names: `student_id`, `math_score`, `date`, `city`.

Pandas exists to add two things on top of NumPy:

1. **Labels** — rows and columns have names (an "index"), not just positions
2. **Relational/set operations** — combining, filtering, and grouping labeled tables, borrowed from **set theory** and **relational algebra** (the math behind SQL and databases)

| Math idea | Pandas concept |
|---|---|
| Vector (labeled) | `Series` |
| Matrix (labeled rows & columns) | `DataFrame` |
| Set membership / predicate | Boolean mask filtering (`df[df.col > 5]`) |
| Set union / intersection / difference | `merge` with `how='outer'/'inner'` |
| Partition of a set + function per part | `groupby(...).agg(...)` |
| Summation notation `Σ` | `.sum()`, `.mean()`, aggregation functions |
| Covariance / correlation matrix | `.cov()`, `.corr()` |
| Cross-tabulation / matrix reshape | `pivot_table`, `pivot` |

Under the hood, a Pandas `DataFrame` **is** a NumPy 2-D array (or several) with an index and column labels bolted on — everything you learned about matrices still applies.

---

## 2. Series & DataFrame ↔ Vector & Matrix

A **Series** is a labeled vector: values + an index. A **DataFrame** is a labeled matrix: rows (index) × columns.

```python
import pandas as pd
import numpy as np

# Series = labeled vector
scores = pd.Series([85, 70, 90, 60], index=["Asha", "Ravi", "Meera", "Kabir"])
print(scores)

# DataFrame = labeled matrix
df = pd.DataFrame({
    "math":    [85, 70, 90, 60],
    "science": [90, 65, 88, 72],
    "english": [78, 80, 95, 68],
}, index=["Asha", "Ravi", "Meera", "Kabir"])
print(df)

# The underlying NumPy matrix is always accessible:
print(df.values)          # plain 2-D ndarray, shape (4, 3)
print(df.values.shape)
```

**Why it matters:** every NumPy operation you learned (mean, std, matrix multiply, `@`) works on `df.values`. Pandas is a *convenience layer* over the exact same linear algebra — with row/column names attached so `df["math"]` means something to a human.

---

## 3. Filtering ↔ Set Theory

In math, a filter is a **predicate** applied to a set: `{x ∈ S | P(x)}` — "all `x` in `S` such that `P(x)` is true." Pandas boolean masking is exactly this.

```python
# WORKED EXAMPLE
high_math = df[df["math"] > 75]
print(high_math)

# Combine predicates: AND (&), OR (|)  -- like set intersection/union
strong_students = df[(df["math"] > 75) & (df["science"] > 75)]
print(strong_students)
```

**Why it matters:** a boolean mask (`df["math"] > 75`) is itself a Series of `True`/`False` — this is the indicator function of a set from math: `1` if the element belongs, `0` if not.

---

## 4. GroupBy ↔ Set Partitions + Aggregation

**Math idea:** a **partition** of a set splits it into non-overlapping subsets that together cover the whole set. `groupby` partitions rows by a shared column value, then applies a summation-style function (`Σ`) to each partition.

```python
sales = pd.DataFrame({
    "region": ["North", "North", "South", "South", "East"],
    "amount": [100, 150, 200, 50, 300],
})

# WORKED EXAMPLE
grouped = sales.groupby("region")["amount"].sum()
print(grouped)
# North: 100+150=250, South: 200+50=250, East: 300
```

Formally, if `Sᵣ` is the subset of rows where `region = r`, this computes `Σ_{i ∈ Sᵣ} amountᵢ` for every group `r` — exactly the same `Σ` notation from the NumPy stats section, just applied per-partition instead of over the whole array.

```python
# Multiple aggregations at once
summary = sales.groupby("region")["amount"].agg(["sum", "mean", "count"])
print(summary)
```

---

## 5. Merge / Join ↔ Relational Algebra

**Math idea:** relational algebra (the theory behind SQL) defines how to combine two sets of labeled tuples. Pandas `merge` implements these operations directly, and they map to set operations on the *matching keys*.

```python
students = pd.DataFrame({
    "student_id": [1, 2, 3],
    "name": ["Asha", "Ravi", "Meera"],
})
grades = pd.DataFrame({
    "student_id": [1, 2, 4],
    "grade": ["A", "B", "A"],
})

# WORKED EXAMPLE
inner = pd.merge(students, grades, on="student_id", how="inner")   # keys in BOTH (set intersection)
left  = pd.merge(students, grades, on="student_id", how="left")    # all of `students`, matched where possible
outer = pd.merge(students, grades, on="student_id", how="outer")   # keys in EITHER (set union), NaN where missing

print(inner)
print(left)
print(outer)
```

| Join type | Set operation on keys |
|---|---|
| `inner` | intersection: `{keys in A} ∩ {keys in B}` |
| `outer` | union: `{keys in A} ∪ {keys in B}` |
| `left` | all of A, matched with B where possible |
| `right` | all of B, matched with A where possible |

---

## 6. Descriptive Statistics & Covariance/Correlation Matrices

Every stat function you learned in NumPy (`mean`, `std`, `var`) applies column-wise on a DataFrame automatically.

```python
# WORKED EXAMPLE
print(df.mean())          # mean of each column
print(df.std())           # std of each column
print(df.describe())      # count, mean, std, min, quartiles, max -- all at once
```

**Covariance matrix** (`Σ` in statistics notation, not to be confused with summation `Σ`): tells you how every pair of columns varies together. **Correlation matrix** is the same idea, normalized to `[-1, 1]`.

```python
cov_matrix = df.cov()
corr_matrix = df.corr()
print(cov_matrix)
print(corr_matrix)
```

**Why it matters:** this `cov_matrix` is *exactly* the input to `np.linalg.eig()` from the NumPy eigenvalues section — the covariance matrix's eigenvectors are the principal components in PCA. Pandas computes the matrix; NumPy (or scikit-learn) decomposes it. This is the direct bridge between the two modules.

---

## 7. Pivot Tables ↔ Matrix Reshaping / Cross-Tabulation

**Math idea:** a pivot table is a **cross-tabulation** — turning long-format data (one row per observation) into a matrix where rows and columns are two different categorical variables, and cells are an aggregated value. This is the tabular equivalent of reshaping a tensor.

```python
records = pd.DataFrame({
    "student": ["Asha", "Asha", "Ravi", "Ravi"],
    "subject": ["math", "science", "math", "science"],
    "score":   [85, 90, 70, 65],
})

# WORKED EXAMPLE
pivoted = records.pivot(index="student", columns="subject", values="score")
print(pivoted)
# Now student x subject is a proper matrix, same shape idea as NumPy's 2-D arrays
```

---

## 8. Quick Reference Cheat Sheet

```python
import pandas as pd

# Creation
pd.Series([1,2,3])
pd.DataFrame({"col1": [...], "col2": [...]})
pd.read_csv("file.csv")

# Inspect (labeled matrix -> plain matrix)
df.shape; df.columns; df.index
df.values                      # underlying NumPy array
df.dtypes

# Filtering (set predicate)
df[df["col"] > 5]
df[(df["a"] > 1) & (df["b"] < 10)]

# GroupBy (partition + aggregate, Σ per group)
df.groupby("col")["value"].sum()
df.groupby("col")["value"].agg(["sum", "mean", "count"])

# Merge/Join (relational algebra / set ops on keys)
pd.merge(a, b, on="key", how="inner")   # intersection
pd.merge(a, b, on="key", how="outer")   # union
pd.merge(a, b, on="key", how="left")
pd.merge(a, b, on="key", how="right")

# Stats
df.mean(); df.std(); df.var(); df.describe()
df.cov()     # covariance matrix -> feeds into PCA (np.linalg.eig)
df.corr()    # correlation matrix

# Reshape (cross-tabulation)
df.pivot(index=..., columns=..., values=...)
df.pivot_table(index=..., columns=..., values=..., aggfunc="mean")
```

---

**Next up:** `matplotlib/MATPLOTLIB_MATH_CONCEPTS.md` — visualizing the vectors, distributions, and functions we've been computing (plotting `y = f(x)`, histograms as discretized probability distributions, scatter plots as 2-D vector spaces).

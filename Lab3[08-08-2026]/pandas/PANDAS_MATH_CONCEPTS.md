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
| Partial function / three-valued logic | `NaN`, `isnull()`, `dropna()`, `fillna()` |

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

## 8. Missing Data ↔ Partial Functions & Three-Valued Logic

**Math idea:** think of a DataFrame column as a function from the row index to a value: `f: index → value`. A **total function** is defined for every element of its domain. A **partial function** is defined only on *part* of its domain — for the rest, `f(i)` simply has no value. That's exactly what a missing entry is: the row exists (the index is defined), but the mapping to a value is not.

This connects to the same relational theory behind `merge` in Section 5: SQL and Pandas both effectively use a **third truth value** — `UNKNOWN` — alongside `True`/`False`, for anywhere a value is missing. That's why `NaN == NaN` evaluates to `False`: "is an unknown value equal to another unknown value?" is itself unknown, not true.

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    "student": ["Asha", "Ravi", "Meera", "Kabir"],
    "math":    [85, np.nan, 90, 60],
    "science": [90, 65, np.nan, 72],
})

# WORKED EXAMPLE
print(np.nan == np.nan)   # False -- "unknown = unknown" is still unknown, not true
print(df.isnull())        # indicator function, same idea as the boolean mask in Section 3
print(df.isnull().sum())  # count of undefined entries per column (per function)
```

**Why it matters:** `df["math"] == np.nan` will *never* match anything, because equality is the wrong tool for expressing "this value doesn't exist." `isnull()` is a dedicated predicate for domain membership — the same indicator-function idea as Section 3's `df["math"] > 75`, just asking "is this element in the defined subset?" instead of "does this element satisfy `P(x)`?"

### Restricting the domain: `dropna()`

If `f` is a partial function on index set `I`, `dropna()` produces the **restriction** of the DataFrame to the largest subset of `I` where `f` is totally defined: `I' = {i ∈ I | f(i) is defined}` (across all columns, or a chosen subset via `subset=`).

```python
# WORKED EXAMPLE
complete_only = df.dropna()                  # restrict to rows fully defined on every column
complete_math = df.dropna(subset=["math"])   # restrict to rows where math specifically is defined
print(complete_only)
print(complete_math)
```

### Extending the domain: `fillna()`

`fillna()` does the opposite of `dropna()` — it takes the partial function and **extends** it to a total function by supplying a value everywhere it was previously undefined. *Which* value you extend with matters mathematically: filling with the column mean preserves the sample mean of the originally-defined values, while filling with 0 or a placeholder label doesn't preserve any statistical property — it's just a convention.

```python
# WORKED EXAMPLE
df_filled = df.copy()
df_filled["math"] = df_filled["math"].fillna(df_filled["math"].mean())
print(df_filled)
```

**Why it matters:** aggregations (`.sum()`, `.mean()` — Section 6) default to `skipna=True`, meaning they silently compute `Σ` over only the *defined* subset of the domain. This is the same "sum over a subset" idea as `groupby` in Section 4 — except here the subset is "where the function is defined," not "where a category matches."

```python
print(df["math"].sum())               # 235.0 -- sums only the 3 defined values
print(df["math"].sum(skipna=False))   # NaN -- undefined "poisons" the sum, like adding an unknown
```

---

## 9. Quick Reference Cheat Sheet

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

# Missing data (partial function <-> total function)
df.isnull(); df.isnull().sum()          # indicator of undefined entries
df.dropna()                              # restrict to fully-defined rows
df.dropna(subset=["col"])                # restrict on specific column(s)
df["col"].fillna(value)                  # extend to a total function with a constant
df["col"].fillna(df["col"].mean())       # extend using a statistic
df["col"].sum(skipna=False)              # let undefined values propagate instead of skipping
```

---

**Next up:** `matplotlib/MATPLOTLIB_MATH_CONCEPTS.md` — visualizing the vectors, distributions, and functions we've been computing (plotting `y = f(x)`, histograms as discretized probability distributions, scatter plots as 2-D vector spaces).

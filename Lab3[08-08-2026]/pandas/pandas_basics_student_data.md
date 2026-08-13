# Pandas Basics: Student Academic Records

A single running example — synthetic student academic data — carried through data generation, first-look exploration, and null handling. Each section has working code plus a one-line talking point for class.

---

## 0. Setup

```python
import pandas as pd
import numpy as np

np.random.seed(42)  # reproducible results 
```



---

## 1. Generate the synthetic dataset

We'll build a small "exam records" table: one row per student per subject, with marks, attendance, and a lab grade — some values deliberately left missing so there's something real to clean later.

```python
n = 200  # number of records

# --- value pools ---
first_names = ["Ananya", "Priya", "Sneha", "Divya", "Meghana", "Lasya", "Keerthi",
                "Harika", "Sowmya", "Nikitha", "Pallavi", "Ramya", "Sindhu", "Varsha"]
last_names = ["Reddy", "Rao", "Sharma", "Naidu", "Prasad", "Kumar", "Chowdary", "Goud"]
branches = ["CSE", "IT", "ECE", "EEE"]
subjects = ["Cloud Computing", "Machine Learning", "DBMS", "Operating Systems", "Data Structures"]

# --- build columns ---
names = [f"{np.random.choice(first_names)} {np.random.choice(last_names)}" for _ in range(n)]
student_ids = [f"21071A05{str(i).zfill(2)}" for i in range(1, n + 1)]

df = pd.DataFrame({
    "student_id": student_ids,
    "name": names,
    "branch": np.random.choice(branches, size=n, p=[0.5, 0.25, 0.15, 0.1]),
    "semester": np.random.choice([3, 4, 5, 6], size=n),
    "subject": np.random.choice(subjects, size=n),
    "internal_marks": np.random.randint(10, 31, size=n).astype(float),   # out of 30
    "external_marks": np.random.randint(25, 71, size=n).astype(float),  # out of 70
    "attendance_percentage": np.round(np.random.uniform(55, 100, size=n), 1),
    "lab_grade": np.random.choice(["A", "B", "C", "D"], size=n),
})

# --- inject realistic missing values ---
# (e.g. external marks missing = student hasn't taken the end-sem exam yet,
#  lab_grade missing = lab evaluation not yet submitted)
rng = np.random.default_rng(7)
df.loc[rng.choice(n, 15, replace=False), "external_marks"] = np.nan
df.loc[rng.choice(n, 10, replace=False), "attendance_percentage"] = np.nan
df.loc[rng.choice(n, 20, replace=False), "lab_grade"] = None
```


---

## 2. First look at the data

```python
df.head()        # first 5 rows — quick sanity check on structure and values
df.head(10)      # or however many rows you want to eyeball
df.shape         # (rows, columns) — (200, 9)
df.columns       # column names
df.dtypes        # data type of each column
df.info()        # dtypes + non-null counts + memory, all in one call
```


---

## 3. Finding null values

```python
df.isnull()                 # same-shape DataFrame of True/False
df.isnull().sum()           # count of nulls per column — the one you'll use most
df.isnull().sum().sum()     # total nulls in the whole DataFrame
df.isnull().mean() * 100    # % missing per column — useful for deciding drop vs fill
```

Expected output shape for `df.isnull().sum()`:

```
student_id                0
name                      0
branch                    0
semester                  0
subject                   0
internal_marks            0
external_marks           15
attendance_percentage    10
lab_grade                20
dtype: int64
```



---

## 4. Handling null values

Two broad strategies: **drop** the incomplete rows, or **fill** the gaps. Which one is right depends on how much data you can afford to lose.

### 4a. Drop

```python
df.dropna()                                  # drops ANY row with at least one null (aggressive)
df.dropna(subset=["external_marks"])         # only drop rows missing external_marks
df.dropna(how="all")                         # only drop rows where EVERY value is null (rare)
df.dropna(thresh=7)                          # keep rows with at least 7 non-null values
```

⚠️ `df.dropna()` on this dataset drops rows for **any** missing column, so it removes far more rows than dropping on a single column — worth showing both side by side so students see the difference.

### 4b. Fill

```python
df_clean = df.copy()  # work on a copy, keep the original for comparison

# numeric columns — fill with a summary statistic
df_clean["external_marks"] = df_clean["external_marks"].fillna(df_clean["external_marks"].mean())
df_clean["attendance_percentage"] = df_clean["attendance_percentage"].fillna(
    df_clean["attendance_percentage"].median()
)

# categorical column — fill with a meaningful label, not a statistic
df_clean["lab_grade"] = df_clean["lab_grade"].fillna("Not Graded")

df_clean.isnull().sum()   # confirm: all zeros now
```



---

## 5. A bit more exploration (optional, time permitting)

```python
df.describe()                        # count, mean, std, min/max, quartiles — numeric columns only
df["branch"].value_counts()          # frequency count of each category
df["subject"].value_counts(normalize=True) * 100   # as percentages

# boolean filtering
low_attendance = df[df["attendance_percentage"] < 65]
cse_students = df[df["branch"] == "CSE"]
cse_low_marks = df[(df["branch"] == "CSE") & (df["internal_marks"] < 15)]
```


---



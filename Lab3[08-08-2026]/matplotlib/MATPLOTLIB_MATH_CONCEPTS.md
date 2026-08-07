# Matplotlib ↔ Math Concepts

**Audience:** Absolute beginners (comfortable with NumPy arrays and Pandas DataFrames)
**Goal:** Understand *why* Matplotlib is built the way it is by connecting each chart type to the math/data idea it visualizes.

---

## 1. Why Matplotlib exists

Every array or table we've built with NumPy and Pandas is a set of numbers. Numbers on their own are hard to reason about — a human can't "see" that a dataset is skewed, that two variables correlate, or that a function is curving upward just by staring at a table of values.

Matplotlib exists to turn numeric structures into **pictures that map 1:1 onto the math**:

| Data/Math idea | Matplotlib concept |
|---|---|
| A function `y = f(x)` | Line plot (`plt.plot`) |
| A point in 2-D space `(x, y)` | Scatter plot (`plt.scatter`) |
| A discretized probability distribution | Histogram (`plt.hist`) |
| A categorical aggregation (groupby result) | Bar chart (`plt.bar`) |
| The five-number summary (min, Q1, median, Q3, max) | Box plot (`plt.boxplot`) |
| A matrix / 2-D grid of values | Heatmap / `imshow` |
| A figure with multiple related plots | Subplots (`plt.subplots`) |

Every chart in Matplotlib is drawn on a **Figure** (the canvas) containing one or more **Axes** (the actual plot area with its own x/y coordinate system). This maps directly onto the math idea of a coordinate plane.

---

## 2. Synthetic Data Generation (why `rng.normal(...)` shows up everywhere below)

**What it is:** synthetic data is data you *generate with code* instead of collecting from the real world — fake-but-realistic numbers that follow a distribution you choose on purpose. From Section 4 onward, nearly every example below starts with a line like `rng.normal(...)` to invent a dataset before plotting it.

**Why we do it (instead of loading a real CSV):**
1. **No dataset required to practice a chart type.** You can invent data with exactly the shape, size, or relationship you need to demonstrate a concept.
2. **You control the "ground truth."** Because *you* pick the mean/spread/relationship when generating the data, you can visually confirm the chart shows what it's supposed to (e.g. "I generated this with mean 50 — does the histogram look centered there?").
3. **Reproducibility.** Real-world randomness can't be replayed. Code-generated randomness can, if you fix the **seed**.

**The tool: NumPy's `Generator` object (`np.random.default_rng`)**

```python
rng = np.random.default_rng(0)   # 0 is the "seed" — any fixed integer works
```

A `Generator` produces pseudo-random numbers — they *look* random but are fully determined by the seed. Same seed → same numbers, every time the code reruns. That's why the examples below create `rng` once and reuse it, instead of calling `np.random.default_rng()` fresh (no seed) each time.

**The distributions used in this file:**

| Call | Produces | Used for |
|---|---|---|
| `rng.normal(loc, scale, size)` | Bell-curve-shaped data around a mean (`loc`) with a spread (`scale`) | Exam scores, heights, salaries — most real measurements cluster this way |
| `rng.uniform(low, high, size)` | Every value in `[low, high)` equally likely | Study hours, arrival times — things with no natural "typical" value |

```python
demo = rng.normal(loc=50, scale=10, size=5)
print(demo)   # 5 synthetic values ~ Normal(mean=50, std=10)
```

**Why it matters:** every chart type in this file (scatter, histogram, box plot, heatmap) is demonstrated on synthetic data first — the same technique you'll use to sanity-check your own plotting code before pointing it at a real dataset.

---

## 3. The Figure/Axes model

```python
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots()   # fig = canvas, ax = one coordinate plane
ax.plot([1, 2, 3], [1, 4, 9])
plt.show()
```

**Why it matters:** `plt.plot(...)` (the "pyplot" shortcut) implicitly creates a figure/axes for you — fine for quick scripts. `fig, ax = plt.subplots()` gives you the explicit objects, which is the pattern you need the moment you want more than one plot, or want to customize axis-by-axis. Both draw on the same underlying model: a coordinate plane, exactly like the `(x, y)` plane from algebra.

---

## 4. Line plots — `y = f(x)`

**Math idea:** a line plot connects a sequence of `(x, y)` points, approximating a continuous function `y = f(x)` by sampling it at discrete `x` values — the same idea as a Riemann sum grid, just for visualization instead of area.

```python
x = np.linspace(0, 2 * np.pi, 100)   # 100 sample points
y = np.sin(x)

plt.plot(x, y)
plt.xlabel("x (radians)")
plt.ylabel("sin(x)")
plt.title("y = sin(x)")
plt.show()
```

**Why it matters:** every "loss curve" you'll see when training a model, every time series, every trend line — all of it is this exact pattern: sample a function or a process at points, connect the dots.

---

## 5. Scatter plots — points in a vector space

**Math idea:** a scatter plot places each `(x, y)` pair as a literal point in 2-D space — this *is* the geometric interpretation of a 2-D vector we used with NumPy (`np.array([x, y])`). Each row of a 2-column data matrix becomes a dot.

```python
rng = np.random.default_rng(0)
x = rng.normal(0, 1, 50)
y = 2 * x + rng.normal(0, 0.5, 50)   # y correlated with x, plus noise

plt.scatter(x, y)
plt.xlabel("x")
plt.ylabel("y")
plt.title("Correlated data")
plt.show()
```

**Why it matters:** scatter plots are how you *see* correlation before you compute `df.corr()` — and how you visually sanity-check clustering or regression results (points colored by predicted class, a fitted line overlaid on the cloud of points).

---

## 6. Histograms — discretized distributions

**Math idea:** a histogram takes continuous or many-valued data, splits the range into **bins** (intervals), and counts how many values fall in each bin — this is a discrete approximation of a **probability density function**. Taller bars = more probability mass in that region.

```python
data = rng.normal(loc=50, scale=10, size=1000)

plt.hist(data, bins=30, edgecolor="black")
plt.xlabel("value")
plt.ylabel("count")
plt.title("Distribution of data (~Normal(50, 10))")
plt.show()
```

**Why it matters:** this is literally how you check whether data is normally distributed, skewed, or bimodal — a judgment call you'll make constantly before choosing a statistical test or a model.

---

## 7. Bar charts — categorical aggregation

**Math idea:** a bar chart visualizes a function from a **finite set of categories** to a number — exactly the shape of a `df.groupby("category")["value"].sum()` result. Each category gets one bar; height = the aggregated value.

```python
subjects = ["Math", "Science", "English"]
avg_scores = [78.4, 82.1, 75.6]     # e.g. from df.groupby("subject")["score"].mean()

plt.bar(subjects, avg_scores, color=["#4C72B0", "#55A868", "#C44E52"])
plt.ylabel("Average score")
plt.title("Average score per subject")
plt.show()
```

**Why it matters:** this is the standard way to present a `groupby` result — the exact table from the Pandas module becomes a picture a non-technical stakeholder can read in two seconds.

---

## 8. Box plots — the five-number summary

**Math idea:** a box plot draws the **five-number summary** (minimum, Q1, median, Q3, maximum) plus outliers, all in one compact shape. The box spans the interquartile range (IQR = Q3 − Q1); the whiskers extend to the most extreme non-outlier points.

```python
groups = [rng.normal(0, 1, 100), rng.normal(2, 1.5, 100), rng.normal(-1, 0.5, 100)]

plt.boxplot(groups, tick_labels=["A", "B", "C"])
plt.ylabel("value")
plt.title("Spread comparison across groups")
plt.show()
```

**Why it matters:** box plots let you compare the **spread and skew** of several groups side by side — much faster than reading three sets of mean/std numbers.

---

## 9. Heatmaps — visualizing a matrix

**Math idea:** `imshow` renders a 2-D array directly as a grid of colored cells — each matrix entry becomes a pixel/cell whose color encodes its value. This is the most direct possible visualization of a matrix.

```python
matrix = np.array([
    [1.0, 0.8, 0.2],
    [0.8, 1.0, 0.5],
    [0.2, 0.5, 1.0],
])   # e.g. a correlation matrix from df.corr()

plt.imshow(matrix, cmap="viridis", vmin=-1, vmax=1)
plt.colorbar(label="correlation")
plt.xticks(range(3), ["Math", "Science", "English"])
plt.yticks(range(3), ["Math", "Science", "English"])
plt.title("Correlation matrix")
plt.show()
```

**Why it matters:** `df.corr()` and `np.cov(...)` produce matrices — a heatmap is how you spot the strong relationships (bright/dark cells) without scanning a grid of numbers.

---

## 10. Subplots — multiple coordinate planes in one figure

**Math idea:** sometimes you need to compare several plots side by side — a grid of independent coordinate planes sharing one canvas.

```python
fig, axes = plt.subplots(1, 2, figsize=(10, 4))

axes[0].plot(x, np.sin(x))
axes[0].set_title("sin(x)")

axes[1].plot(x, np.cos(x))
axes[1].set_title("cos(x)")

plt.tight_layout()
plt.show()
```

**Why it matters:** comparing "before vs after," or plotting several features against a target, always uses this pattern — one figure, a grid of axes, each holding one comparison.

---

## 11. Quick Reference Cheat Sheet

```python
import matplotlib.pyplot as plt
import numpy as np

# Figure/Axes
fig, ax = plt.subplots()             # one plot
fig, axes = plt.subplots(2, 2)       # grid of plots

# Chart types
plt.plot(x, y)                       # line: y = f(x)
plt.scatter(x, y)                    # points in 2-D space
plt.hist(data, bins=30)              # discretized distribution
plt.bar(categories, values)          # categorical aggregation
plt.boxplot(groups)                  # five-number summary
plt.imshow(matrix, cmap="viridis")   # matrix as a grid of colors

# Labeling
plt.xlabel("..."); plt.ylabel("..."); plt.title("...")
plt.legend()                         # needs label="..." on each plot call
plt.colorbar()                       # for imshow/heatmaps

# Multiple series on one plot
plt.plot(x, y1, label="series 1")
plt.plot(x, y2, label="series 2")
plt.legend()

# Saving & showing
plt.savefig("figure.png", dpi=150, bbox_inches="tight")
plt.show()
```

---

**Next up:** with NumPy (compute), Pandas (organize), and Matplotlib (visualize) in place, you have the full data-engineering toolchain used before any model is trained — the next module (scikit-learn) builds directly on top of this: standardized arrays in, fitted models out, results visualized right back with the tools from this file.

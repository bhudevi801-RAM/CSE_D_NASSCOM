# 🔢 Introduction to NumPy — Definitions & Reference

### For B.Tech II Year CSE Students | AI/ML & Programming Practical

This is the companion reference for `Practice_NumPy.ipynb`. Read the matching section here
**before** running each Demo cell — the notebook itself only has clean code, all the
"why does this exist / how does this show up in ML" explanation lives here.

**Topics covered:** Why NumPy → Creating Arrays → Indexing & Reshaping → Aggregate Functions →
Vectorized Ops & Broadcasting → Dot Product & Matrix Multiplication

**Learning Outcomes (COs):**
- CO1: Explain why NumPy arrays replace Python lists for numerical/ML work.
- CO2: Create, index, slice, and reshape arrays correctly.
- CO3: Use aggregate and vectorized operations instead of manual loops.
- CO4: Compute dot products and matrix multiplications, and connect them to how a neuron
  and a neural network layer actually compute their output.

---

## 1️⃣ Why NumPy Arrays?

**The problem with plain Python lists:** a Python list can hold anything, so under the hood
each element is a full Python object with its own type/memory overhead. Doing math on a list
means writing a loop yourself — `[x * 2 for x in my_list]` — and that loop runs in slow,
interpreted Python, one element at a time.

**NumPy's fix:** a NumPy `array` stores elements of a **single fixed type** in one contiguous
block of memory, and math operations run as compiled, vectorized C code across the whole array
at once — no explicit loop needed, and it can be 10–100x faster on large data.

**Why this matters for ML:** every dataset you'll touch — a batch of images, a table of student
marks, a set of word embeddings — is eventually a NumPy array (or something built directly on
top of it, like a Pandas DataFrame or a PyTorch tensor). Model weights, input features, and
predictions are all arrays. If you don't know how to shape, slice, and combine arrays, you can't
follow what an ML pipeline is actually doing to your data.

**Key properties**
- **Homogeneous** → all elements share one dtype (e.g. all `float64`).
- **Fixed shape** → an array has a defined shape (e.g. `(3, 4)` = 3 rows, 4 columns) that
  operations must respect.
- **Vectorized** → operations apply to the whole array at once, no manual loop.

---

## 2️⃣ Creating & Inspecting Arrays

**Why multiple creation functions?** Real ML code rarely types out every number by hand — you
generate placeholder arrays (`zeros`, `ones`), evenly spaced ranges (`arange`, `linspace`), or
random data for testing a model before real data is ready (`random`).

| Function | Why we use it |
|---|---|
| `np.array(list)` | Convert existing Python data (e.g. a dataset you already have) into a fast array |
| `np.zeros(shape)` | Placeholder array of zeros — e.g. initializing a fresh weight matrix |
| `np.ones(shape)` | Placeholder array of ones — e.g. a bias vector before training |
| `np.arange(start, stop, step)` | An evenly-stepped range — e.g. epoch numbers 0..99 |
| `np.linspace(start, stop, n)` | `n` evenly spaced points between two values — e.g. a smooth x-axis for plotting a function |
| `np.random.rand(shape)` | Random values in `[0, 1)` — e.g. randomly initializing weights before training starts |
| `.shape` | The array's dimensions `(rows, cols, ...)` — you check this constantly to make sure two arrays are compatible before combining them |
| `.dtype` | The data type stored — matters for memory and precision (e.g. `int32` vs `float64`) |
| `.ndim` | Number of dimensions — 1D vector, 2D matrix, 3D+ tensor (e.g. a batch of images) |

---

## 3️⃣ Indexing, Slicing & Reshaping

**Why this matters for ML:** a dataset is usually a 2D array — rows are samples (e.g. students),
columns are features (e.g. marks in each subject). You are constantly pulling out "one row"
(one sample), "one column" (one feature across all samples), or reshaping data into the exact
shape a model function expects.

| Operation | Why we use it |
|---|---|
| `arr[i]` | Get row `i` — e.g. one student's full feature vector |
| `arr[i, j]` | Get a single element — e.g. one student's mark in one subject |
| `arr[:, j]` | Get column `j` across ALL rows — e.g. everyone's marks in one subject |
| `arr[a:b]` | Slice a range of rows — e.g. the first 10 samples of a batch |
| Boolean indexing `arr[arr > x]` | Filter values by a condition — e.g. every mark above 80, without a manual loop |
| `.reshape(new_shape)` | Change the array's shape without changing its data — e.g. flattening a 28×28 image into a 784-length vector for a model input |
| `.flatten()` | Collapse any array down to 1D — common before feeding data into certain model layers |

---

## 4️⃣ Aggregate Functions

**Why this matters for ML:** you rarely want individual numbers — you want the *class average*,
the *column-wise mean* (average of each feature across all samples, used to normalize data
before training), or the *row-wise max*. The `axis` argument controls which direction the
aggregation collapses.

- `axis=0` → collapse **down the rows** → one result **per column**.
- `axis=1` → collapse **across the columns** → one result **per row**.
- No axis → collapse the **entire array** into a single number.

| Function | Why we use it |
|---|---|
| `np.sum(arr, axis=...)` | Total — e.g. total marks per student (axis=1) or per subject (axis=0) |
| `np.mean(arr, axis=...)` | Average — e.g. average mark per subject, used to normalize features before training a model |
| `np.min(arr)` / `np.max(arr)` | Extremes — e.g. the lowest/highest mark anywhere in the dataset |
| `np.std(arr, axis=...)` | Spread of the data — a large std means a feature varies a lot, which affects how a model should treat it |
| `np.argmax(arr, axis=...)` | The **index** of the max value — e.g. which class a model is most confident about (this is exactly how a classifier picks its predicted label) |

---

## 5️⃣ Vectorized Operations & Broadcasting

**Why this matters for ML:** training a model means doing the *same* arithmetic to *every*
sample, over and over, for thousands of iterations. If that were a Python `for` loop, training
would be far too slow. Vectorization is what makes ML computation on real-sized data feasible.

**Broadcasting:** NumPy can combine arrays of *different but compatible* shapes without you
manually looping or copying data — e.g. adding one bias vector `(3,)` to every row of a
`(100, 3)` data matrix happens in one line: `data + bias`. This is exactly how a bias term gets
added across an entire batch in a neural network layer.

| Operation | Why we use it |
|---|---|
| `arr + / - / * / /` (elementwise) | Apply the same arithmetic to every element at once — e.g. rescaling all pixel values by 1/255 |
| Array `+` array (same shape) | Combine two datasets element-by-element — e.g. adding noise to a signal |
| Array `+` smaller array (broadcasting) | Apply a smaller pattern (e.g. one bias per column) across every row automatically |
| Comparison (`arr > x`) | Produces a boolean array — the basis of filtering and of how a model checks predictions against a threshold |

---

## 6️⃣ Dot Product & Matrix Multiplication

**Why this matters for ML — this is the single most important NumPy skill for ML:** a neuron's
output is literally a **dot product**: multiply each input by its matching weight, and sum the
results — `output = np.dot(inputs, weights) + bias`. A neural network **layer** processes many
neurons and many samples at once, and that's a **matrix multiplication** — one matrix of inputs
times one matrix of weights. Every forward pass through a neural network, at its core, is a
sequence of matrix multiplications.

| Operation | Why we use it |
|---|---|
| `np.dot(a, b)` (1D · 1D) | Sum of elementwise products — this IS what a single neuron computes from its inputs and weights |
| `a @ b` (2D matrices) | Matrix multiplication — this IS what an entire layer computes for an entire batch of samples at once |
| Shape rule `(m, n) @ (n, p) → (m, p)` | The inner dimensions must match — this is why you'll constantly check `.shape` before multiplying; a shape mismatch is one of the most common ML bugs |
| `arr.T` (transpose) | Flip rows and columns — often needed to make two matrices' shapes compatible for multiplication |

---

## ✅ Wrap-up

You've now covered the NumPy foundation that every ML library (Pandas, scikit-learn, PyTorch,
TensorFlow) is built on top of:
**Why arrays beat lists → creating/inspecting arrays → indexing & reshaping → aggregation →
vectorization & broadcasting → dot products & matrix multiplication.**

Before moving on, ask yourself:
- Given raw data, can I get it into the right *shape* for a model?
- Do I understand *why* a neuron's computation is just a dot product, and a layer's computation
  is just a matrix multiplication?

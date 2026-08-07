# NumPy ↔ Math Concepts

**Audience:** Absolute beginners (new to Python and to formal math notation)
**Goal:** Understand *why* NumPy is built the way it is by connecting each feature to the math idea behind it.

---

## 1. Why NumPy exists

A plain Python list stores numbers, but it doesn't know any math — `[1,2,3] + [4,5,6]` gives you `[1,2,3,4,5,6]` (list concatenation), not `[5,7,9]` (vector addition).

NumPy exists because AI, data engineering, and scientific computing are built on **linear algebra** — vectors, matrices, and operations on them. NumPy gives Python:

1. A data structure that behaves like a **mathematical vector/matrix** (the `ndarray`)
2. Fast, math-correct operations on that structure
3. A bridge to every other library we'll cover (Pandas, Matplotlib, Scikit-learn, PyTorch all use NumPy arrays underneath)

| Math idea | NumPy concept |
|---|---|
| Scalar | 0-D array |
| Vector | 1-D array |
| Matrix | 2-D array |
| Tensor | N-D array |
| Vector/matrix arithmetic | Element-wise operators (`+`, `-`, `*`, `/`) |
| Dot product / matrix multiplication | `@` operator, `np.dot`, `np.matmul` |
| Mean, variance, std deviation | `np.mean`, `np.var`, `np.std` |
| Eigenvalues / eigenvectors | `np.linalg.eig` |
| Singular Value Decomposition | `np.linalg.svd` |
| System of linear equations | `np.linalg.solve` |

---

## 2. Scalars, Vectors, Matrices, Tensors

In math, we organize numbers by **how many directions (axes)** they need:

- **Scalar** — a single number. `5`
- **Vector** — an ordered list of numbers, e.g. a point in space: `[2, 3]`
- **Matrix** — a grid of numbers (rows × columns): `[[1,2],[3,4]]`
- **Tensor** — the general case, any number of axes (a stack of matrices, etc.)

In NumPy, all four are the *same* object type (`ndarray`) — they just differ in **shape**.

```python
import numpy as np

scalar = np.array(5)              # shape: ()
vector = np.array([2, 3, 4])      # shape: (3,)
matrix = np.array([[1, 2], [3, 4]])  # shape: (2, 2)
tensor = np.array([[[1,2],[3,4]], [[5,6],[7,8]]])  # shape: (2, 2, 2)

print(scalar.shape, vector.shape, matrix.shape, tensor.shape)
print(scalar.ndim, vector.ndim, matrix.ndim, tensor.ndim)
```

**Why it matters for AI/data engineering:** a grayscale image is a matrix (height × width). A color image is a tensor (height × width × 3 color channels). A batch of images fed into a neural network is a 4-D tensor (batch × height × width × channels). Every dataset you'll load is ultimately shaped as one of these.

---

## 3. Vector Operations

A vector represents a **point** or a **direction** with magnitude. NumPy implements the standard vector operations from linear algebra directly.

### 3.1 Addition & scalar multiplication

Math: `v + w`, `c·v`

```python
v = np.array([1, 2, 3])
w = np.array([4, 5, 6])

print(v + w)      # [5 7 9]   -- element-wise addition
print(2 * v)      # [2 4 6]   -- scalar multiplication
```

### 3.2 Dot product

Math: `v · w = Σ vᵢwᵢ` — measures how much two vectors "point the same way." This single operation underlies cosine similarity, projections, and (later) every neural network layer.

```python
dot = np.dot(v, w)          # or v @ w
print(dot)   # 1*4 + 2*5 + 3*6 = 32
```

### 3.3 Norm (length / magnitude)

Math: `‖v‖ = √(Σ vᵢ²)`

```python
length = np.linalg.norm(v)
print(length)   # sqrt(1+4+9) = 3.7416...
```

**Why it matters:** distance between two data points (used in k-NN, clustering) is the norm of their difference: `np.linalg.norm(a - b)`.

---

## 4. Matrix Operations

A matrix represents a **linear transformation** or a **table of data** (rows = samples, columns = features).

### 4.1 Transpose

Math: `Aᵀ` flips rows and columns.

```python
A = np.array([[1, 2, 3], [4, 5, 6]])   # shape (2, 3)
print(A.T)                              # shape (3, 2)
```

### 4.2 Matrix multiplication

Math: `(AB)ᵢⱼ = Σₖ Aᵢₖ Bₖⱼ` — this is *not* element-wise; it's row-times-column dot products.

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

print(A * B)     # element-wise:  [[5,12],[21,32]]  -- NOT matrix multiplication
print(A @ B)     # matrix mult:   [[19,22],[43,50]]
```

⚠️ **Common beginner trap:** `*` is element-wise, `@` (or `np.matmul`) is true matrix multiplication. Every neural network layer computes `output = input @ weights + bias` — this is matrix multiplication, not `*`.

### 4.3 Identity, Inverse, Determinant

```python
I = np.eye(2)                    # identity matrix
A_inv = np.linalg.inv(A)         # inverse: A @ A_inv ≈ I
det = np.linalg.det(A)           # determinant: is A invertible? (det=0 means no)
```

### 4.4 Solving linear systems

Math: solve `Ax = b` for `x` (this is what "solving equations" means in linear algebra — e.g., linear regression's closed-form solution).

```python
A = np.array([[3, 1], [1, 2]])
b = np.array([9, 8])
x = np.linalg.solve(A, b)
print(x)   # the values that satisfy both equations simultaneously
```

---

## 5. Broadcasting (Shapes matter)

Broadcasting is NumPy's rule for how arrays of **different shapes** can still be combined, by mentally "stretching" the smaller one — without actually copying data (efficient!).

Math analogy: adding a constant to every entry of a matrix, or subtracting the mean vector from every row of a data matrix.

```python
matrix = np.array([[1, 2, 3],
                    [4, 5, 6]])          # shape (2, 3)
row_means = matrix.mean(axis=1)          # shape (2,)  -- mean of each row

col_means = matrix.mean(axis=0)          # shape (3,)  -- mean of each column
centered = matrix - col_means            # (2,3) - (3,) broadcasts -> (2,3)
print(centered)
```

**Why it matters:** "mean-centering" data (subtracting the average) is the *first step* of PCA and of standardizing features before feeding them to an ML model. Broadcasting is what makes that a one-line operation instead of a loop.

**Rule of thumb (beginner-friendly):** two shapes are compatible if, comparing dimensions from the right, they're either equal or one of them is 1.

---

## 6. Basic Statistics as NumPy Functions

| Math concept | Formula | NumPy |
|---|---|---|
| Mean (average) | `μ = (1/n)Σxᵢ` | `np.mean(x)` |
| Variance | `σ² = (1/n)Σ(xᵢ-μ)²` | `np.var(x)` |
| Standard deviation | `σ = √variance` | `np.std(x)` |
| Standardization (z-score) | `(x - μ)/σ` | `(x - x.mean()) / x.std()` |

```python
data = np.array([2, 4, 4, 4, 5, 5, 7, 9])
print(data.mean(), data.var(), data.std())

standardized = (data - data.mean()) / data.std()
print(standardized)   # mean ≈ 0, std ≈ 1
```

**Why it matters:** almost every ML model trains better on standardized data. This 1-line formula *is* the math definition of a z-score — nothing hidden.

---

## 7. Eigenvalues, Eigenvectors & SVD (preview — deep dive comes with scikit-learn/PCA)

An **eigenvector** of a matrix `A` is a vector that doesn't change *direction* when transformed by `A` — it only gets scaled by a number called the **eigenvalue** (`λ`).

Math: `A v = λ v`

```python
A = np.array([[2, 0], [0, 3]])
eigenvalues, eigenvectors = np.linalg.eig(A)
print("eigenvalues:", eigenvalues)
print("eigenvectors:\n", eigenvectors)
```

**Singular Value Decomposition (SVD)** generalizes this idea to *any* matrix (not just square ones), decomposing it into three matrices `A = U Σ Vᵀ`.

```python
A = np.array([[1, 2], [3, 4], [5, 6]])   # not square: 3x2
U, S, Vt = np.linalg.svd(A)
print(S)   # the "singular values" -- how much each direction matters
```

**Why it matters:** this is *exactly* the math behind PCA (dimensionality reduction) in scikit-learn, and behind recommendation systems and image compression. We'll build the full intuition when we reach the scikit-learn module — for now, just know NumPy gives you these decompositions as one-liners.

---

## 8. Quick Reference Cheat Sheet

```python
import numpy as np

# Creation
np.array([1,2,3])           # from list
np.zeros((2,3)); np.ones((2,3)); np.eye(3)
np.arange(0, 10, 2)          # like range()
np.linspace(0, 1, 5)         # 5 evenly spaced points

# Shape & structure
a.shape; a.ndim; a.size
a.reshape(2, 3)

# Element-wise math
a + b; a - b; a * b; a / b; a ** 2

# Linear algebra
a @ b or np.matmul(a, b)     # matrix multiplication
np.dot(v, w)                  # dot product
np.linalg.norm(v)             # vector length
a.T                            # transpose
np.linalg.inv(a)              # inverse
np.linalg.det(a)              # determinant
np.linalg.solve(a, b)         # solve Ax = b
np.linalg.eig(a)              # eigenvalues/eigenvectors
np.linalg.svd(a)              # singular value decomposition

# Stats
a.mean(); a.var(); a.std(); a.sum(); a.min(); a.max()
a.mean(axis=0)  # per column
a.mean(axis=1)  # per row
```

---

**Next up:** `pandas/PANDAS_MATH_CONCEPTS.md` — how DataFrames extend this into labeled, tabular data engineering (rows/columns as relational algebra, joins, group-by as aggregation functions).

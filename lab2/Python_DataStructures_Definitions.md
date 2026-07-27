# 🐍 Python Data Structures & File Handling — Definitions & Reference

### For B.Tech II Year CSE Students | AI/ML & Programming Practical

This is the companion reference for `Python_DataStructures_Lab_Manual_STUDENT_ExerciseOnly.ipynb`.
Read the relevant section here **before** running each Demo cell in the notebook — the notebook
itself only has clean code, so all the "why does this exist / when do I use it" explanation
lives here.

**Topics covered:** Lists → Tuples → Dictionaries → Sets → File Handling

**Learning Outcomes (COs):**
- CO1: Select the appropriate data structure (list/tuple/dict/set) for a given real-world problem.
- CO2: Apply core operations on each structure correctly and explain *why* each is used.
- CO3: Read from and write to files safely, including handling missing-file errors.
- CO4: Reason about trade-offs (mutability, ordering, uniqueness, lookup speed) between structures.

**Suggested time budget:** Lists (25 min) · Tuples (20 min) · Dictionaries (25 min) · Sets (25 min) · File Handling (25 min) ≈ 2 hours

---

## 1️⃣ Lists

**Why do Lists exist?**
Imagine storing the marks of 60 students. Creating 60 separate variables (`mark1, mark2, ... mark60`)
would be unmanageable. A **list** stores **multiple values in a single, ordered, mutable container**,
accessible by index.

**Key properties**
- **Ordered** → items keep the position you inserted them in.
- **Mutable** → you can change, add, or remove items after creation.
- **Allows duplicates** → `[1, 1, 2]` is valid.
- Written using square brackets: `[ ]`

**When to reach for a list:** a collection of similar things that may change over time (grow,
shrink, get sorted, get updated) — a task queue, student names, a shopping cart.

**Key functions**

| Function/Operation | Why we use it |
|---|---|
| `append(x)` | Add one new item to the end — a new student joins the class |
| `extend(iterable)` | Merge another list in — adding a whole new batch of students |
| `insert(i, x)` | Add an item at a specific position — a late entry at rank 3 |
| `remove(x)` | Delete an item by value — removing a student named "Ravi" |
| `pop(i)` | Delete by position and return the value — useful for stacks/queues |
| `sort()` | Arrange in order — ranking students by marks |
| `reverse()` | Flip the order — "most recent first" |
| `index(x)` | Find where a value is — the rank of a specific student |
| `count(x)` | Count repeats of a value — how many students scored exactly 90 |
| `len()` | Total size — total number of students |
| slicing `[a:b]` | Extract a portion — top 5 rankers only |

**Complexity note:** `append()`/`pop()` (from the end) are O(1). `insert()`, `remove()`, and
`pop(i)` (from the middle) are O(n) — Python has to shift every following element. Keep this
in mind when comparing lists to sets later.

**🧠 Brainstorm Challenge — Lists**
> Your college wants a daily attendance roll-call system for a class of 60 students, where:
> new students can join anytime, a "dropped out" student must be removed immediately, and at
> semester end the faculty wants the 3 students with the least classes attended (attendance
> count stored as a separate list, same order as the names list).
>
> 1. Which list operations would you use for each requirement, and why?
> 2. Is a list a good or bad choice here compared to the other structures in this reference?
>    (Revisit after finishing Sets.)

---

## 2️⃣ Tuples

**Why do Tuples exist?**
Suppose you store `(latitude, longitude)` of your college — it should never be accidentally
modified. A **tuple** is like a list but **immutable**: a deliberate safety feature for data
that should never change.

**Key properties**
- **Ordered** → same as list.
- **Immutable** → cannot append/remove/modify after creation.
- Written using round brackets: `( )`
- Slightly faster and uses less memory than a list (no resizing bookkeeping needed).

**When to reach for a tuple:** a fixed, related group of values that shouldn't change —
coordinates `(x, y)`, an RGB color `(255, 0, 0)`, a database record `(roll_no, name, dept)`,
or when you need the collection as a **dictionary key** (lists can't be keys — see hashing below).

**Key functions/operations**

| Function/Operation | Why we use it |
|---|---|
| `count(x)` | Count occurrences — how many times a grade appears in a fixed record |
| `index(x)` | Find position of a value — locate "CSE" in a fixed `(roll, name, dept)` record |
| Packing `t = 1, 2, 3` | Bundle multiple values into one variable — returning multiple values from a function |
| Unpacking `a, b, c = t` | Split a tuple into individual variables — common with function returns |
| `len()` | Number of fields in the fixed record |
| Using as dict key | Tuples are immutable, so Python can hash them — lists cannot be keys |

**🧠 Brainstorm Challenge — Tuples**
> You are designing a system to store GPS check-in points for a college trekking trip — each
> point is a fixed `(latitude, longitude, altitude)`. Once recorded, no team member's app should
> be able to edit it (to prevent tampering).
>
> 1. Why is a tuple a better structural choice than a list for the "no accidental editing" requirement?
> 2. If a checkpoint genuinely needs correcting (a GPS glitch), tuples can't be edited in place —
>    how would you "fix" one entry in a collection of tuples? (Think: building a new tuple/list.)

---

## 3️⃣ Dictionaries

**Why do Dictionaries exist?**
Looking up a word in a real dictionary, you jump straight to the word rather than scanning every
page. A **dictionary** stores data as **key–value pairs**, letting you fetch a value instantly
by a meaningful key (e.g. `marks["Ravi"]`) instead of remembering a numeric position.

**Key properties**
- Stores data as `key: value` pairs.
- Keys must be **unique** and **immutable** (strings, numbers, tuples — not lists).
- **Mutable** → values can be added, updated, or removed.
- From Python 3.7+, dictionaries **preserve insertion order**.
- Written using curly braces: `{ }`

**When to reach for a dictionary:** fast lookup by a meaningful label — student-name → marks,
word → meaning, roll-number → attendance percentage, product-name → price.

**Key functions**

| Function/Operation | Why we use it |
|---|---|
| `dict[key] = value` | Add or update an entry — record/update a student's mark |
| `get(key, default)` | Safely fetch a value without crashing if the key doesn't exist |
| `keys()` | All labels — e.g., all student names |
| `values()` | All data — all marks, useful for `sum()`/`max()` |
| `items()` | Key-value pairs together — the standard way to loop through a dict |
| `pop(key)` | Remove an entry by key — a student who withdrew from the exam |
| `update(other_dict)` | Merge another dict in — add a new batch's marks in bulk |
| `in` keyword | Check existence without error — "has this student already been graded?" |

**🧠 Brainstorm Challenge — Dictionaries**
> Your bootcamp wants a student performance tracker where each student (key) maps to a
> dictionary of subject-wise marks (value): `{"Anika": {"Python": 88, "Maths": 76}, ...}`
>
> 1. Why might this nested-dictionary structure be a better fit than 3 separate flat
>    dictionaries (`python_marks`, `maths_marks`, `names`)?
> 2. How would you find the single highest mark across all students and subjects, and by whom?
>    Describe your approach — which dictionary methods would you combine, and in what order?

---

## 4️⃣ Sets

**Why do Sets exist?**
Suppose 200 students register for 3 workshops, some registering for more than one. How do you
quickly find how many *unique* students registered overall, or which students registered for
*both* Workshop A and B? A **set** stores only unique values and gives fast built-in support for
mathematical set operations — union, intersection, difference.

**Key properties**
- **Unordered** → no guaranteed index/position, so no slicing.
- **No duplicates** → adding an existing value has no effect.
- **Mutable** → items can be added/removed, but items themselves must be immutable (no lists
  inside a set).
- Written using curly braces without colons: `{ }` — an empty set must be `set()`, not `{}`
  (`{}` creates an empty dictionary!).

**When to reach for a set:** removing duplicates instantly, or answering "common between two
groups", "only in group A", "everything combined without repeats".

**Key functions**

| Function/Operation | Why we use it |
|---|---|
| `add(x)` | Insert one unique value — duplicates are silently ignored |
| `update(iterable)` | Merge many values in at once, duplicates auto-removed |
| `remove(x)` / `discard(x)` | Delete a value (`remove` errors if missing, `discard` doesn't) |
| `union(other)` / `\|` | Combine two groups completely — all students in either workshop |
| `intersection(other)` / `&` | Find what's common — students registered for both workshops |
| `difference(other)` / `-` | Find what's exclusive — students only in Workshop A |
| `issubset(other)` | Check if one group is entirely contained in another |
| `in` keyword | Extremely fast membership check — much faster than `in` on a list for large data |

**Complexity note — why sets are "fast":** checking `x in a_set` is O(1) on average because sets
use hashing internally, while checking `x in a_list` is O(n) because Python scans element by
element. For the 200-student, 3-workshop example, checking membership against a list is far
slower than against a set once the numbers get large. This is the same idea behind hash tables.

**🧠 Brainstorm Challenge — Sets**
> Your bootcamp runs 4 workshops in a week. A student may register for multiple workshops, and
> organizers want to print an ID card only once per student (no duplicates), plus a report of
> students who registered for 3 or more workshops ("super-active" students).
>
> 1. Why does using a set instead of a list automatically solve the "print ID card only once" problem?
> 2. With 4 separate sets (one per workshop), describe how you'd find students appearing in 3
>    or more of those 4 sets. (Hint: pairwise intersections, or counting occurrences another way.)

---

## 5️⃣ File Handling

**Why does File Handling exist?**
Every variable and data structure disappears the moment your program ends (volatile memory).
**File handling** lets a Python program read from and write to disk, so data becomes
**persistent** and **shareable**.

**When to reach for file handling:** whenever data must survive after the program stops —
logs, reports, datasets, configuration.

**The `with open(...) as f:` pattern:** always prefer `with open(filename, mode) as f:` over
manually calling `open()`/`close()` — `with` automatically closes the file even if an error
occurs inside the block. Forgetting to close a file can cause data corruption or "file in use"
errors.

**Key functions/modes**

| Function/Mode | Why we use it |
|---|---|
| `open(file, "w")` | Write mode — creates a new file OR completely overwrites an existing one |
| `open(file, "a")` | Append mode — adds new content at the end without deleting existing content |
| `open(file, "r")` | Read mode (default) — only reads, can't accidentally modify the file |
| `f.write(text)` | Write a string to the file — logging one line of output |
| `f.writelines(list)` | Write multiple lines at once from a list — saving all records together |
| `f.read()` | Read the entire file as one string — good for small files |
| `f.readline()` | Read just one line at a time — useful for huge files processed line-by-line |
| `f.readlines()` | Read all lines into a list of strings — good for looping/processing each line |
| `with open(...) as f:` | Automatically closes the file, even if an error occurs |

**Handling a missing file (`try`/`except`):** reading a file that doesn't exist raises
`FileNotFoundError` and crashes the program. In real applications (e.g. a lab-server app reading
a config or log file), this is a common bug — handle it explicitly instead of crashing.

**🧠 Brainstorm Challenge — File Handling**
> You're building a login-attempt logger for a college lab-server application: every login
> (success or failure) adds one line to `login_log.txt`, and old entries must never be deleted,
> even if the program restarts a hundred times a day.
>
> 1. Which file mode (`"w"`, `"a"`, `"r"`) is safe for every login event, and why would the
>    wrong mode be a serious bug here?
> 2. At day's end, the admin wants the count of failed logins. Describe how you'd read the log
>    and compute this — which read function (`read()`, `readline()`, `readlines()`) fits best, and why?

---

## ✅ Wrap-up

You've now covered the 5 core building blocks every Python programmer uses daily:
**Lists, Tuples, Dictionaries, Sets, and File Handling.**

Before moving on, ask yourself:
- Given a new problem, can I identify which of these 5 structures fits best?
- Do I understand *why* each function exists, not just its syntax?

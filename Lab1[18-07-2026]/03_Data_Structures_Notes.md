# Lab 2 Notes — Lists, Tuples, Sets & Dictionaries (CSE Student Academic & Coding Profile System)

Lab 1 introduced lists and dictionaries briefly. This lab slows down and asks: **what exactly is different between a list, a tuple, a set, and a dictionary — and how do you pick the right one?** Every example uses something you already deal with every semester: subjects, exam slots, programming languages, and gradebooks.

---

## The Core Comparison

| | List | Tuple | Set | Dictionary |
|---|---|---|---|---|
| Syntax | `["DSA", "OS"]` | `("DSA", "10:00 AM")` | `{"Python", "C"}` | `{"DSA": 92}` |
| Ordered? | Yes | Yes | No | Yes (insertion order, Python 3.7+) |
| Changeable (mutable)? | Yes | **No** | Yes | Yes |
| Duplicates allowed? | Yes | Yes | **No** | Keys: no. Values: yes |
| Accessed by | Index (`subjects[0]`) | Index (`slot[0]`) | Membership (`in`), not index | Key (`gradebook["DSA"]`) |
| Best for | A subject list that changes each semester | A fixed record that shouldn't change | Unique skills / fast membership checks | Looking something up by name |

Keep this table in mind — every section below justifies one row of it.

---

## 1. Lists — Subjects Enrolled This Semester

A list is ordered and changeable — exactly what a semester's subject list needs, since you add electives and drop backlogs during registration.

```python
enrolled_subjects = ["Data Structures", "OS", "DBMS"]
enrolled_subjects.append("Computer Networks")
enrolled_subjects[0] = "DSA (Advanced)"      # lists let you overwrite an item by position
print(enrolled_subjects)
```

**Useful list methods:**

```python
lab_submissions = ["DSA", "OS", "DSA", "DBMS", "OS", "OS"]   # one entry per submission made

lab_submissions.sort()
print(lab_submissions)

print(lab_submissions.count("OS"))       # how many OS lab submissions made -> 3
print(len(lab_submissions))                # total submissions made

recent_3 = lab_submissions[-3:]             # slicing - last 3 submissions
print(recent_3)
```

Notice `lab_submissions` has **repeats** ("OS" appears 3 times) — a list is fine with that, since every submission is a separate event. Keep this in mind for Section 3.

---

## 2. Tuples — A Record That Should Never Change

A tuple looks like a list but is **immutable** — once created, it can't be changed. That's not a limitation, it's the *point*: some data genuinely shouldn't be editable by accident.

**Example:** A published exam slot — a `(subject, time)` pair that a scheduling script should never be able to silently overwrite.

```python
exam_slot = ("Data Structures", "10:00 AM")

subject, time = exam_slot        # tuple unpacking
print(f"{subject} exam is at {time}")

# exam_slot[1] = "11:00 AM"      # <- this line would raise TypeError: tuples don't support item assignment
```

**Why this matters:** imagine a script looping through exam slots to "adjust" a clashing time and it accidentally does `slot[1] = new_time` on what it assumed was a list — if `exam_slot` were a list, that bug would silently corrupt the published timetable. As a tuple, Python stops it immediately with an error, at the exact line it happened.

```python
student_identity = ("21A91A0542", "CSE")     # roll number + branch - fixed once allotted
print(student_identity)
```

**Rule of thumb:** if the data is a fixed record (an exam slot, a roll number, a `(subject, credits)` pairing) — use a tuple. If it's a working collection you'll add to/remove from — use a list.

---

## 3. Sets — Unique Skills Only

A set is unordered and **automatically removes duplicates**. It also supports fast membership checks and mathematical set operations.

**Example:** From Section 1's `lab_submissions` list (which has repeats), find the **unique subjects submitted this week**:

```python
lab_submissions = ["DSA", "OS", "DSA", "DBMS", "OS", "OS"]
unique_subjects_submitted = set(lab_submissions)
print(unique_subjects_submitted)     # {'DSA', 'OS', 'DBMS'} - duplicates gone, order not guaranteed
```

**Set operations — comparing two students' coding skills:**

```python
your_skills = {"Python", "C", "Git", "MySQL"}
friend_skills = {"Python", "Java", "Git", "React"}

print(your_skills & friend_skills)   # intersection: skills you BOTH know -> {'Python', 'Git'}
print(your_skills | friend_skills)   # union: combined skillset across both of you
print(your_skills - friend_skills)   # difference: skills only YOU know -> {'C', 'MySQL'}
```

**Why not just use a list here?** Checking `"Python" in your_skills` is fast regardless of set size. It also makes "which skills do we have in common" a single operator (`&`) instead of a manual loop comparing every item in one list against every item in another.

---

## 4. Dictionaries — Your Gradebook

A dictionary maps a key to a value — Lab 1 used this for a lookup table. This lab goes one level deeper: **nested dictionaries** and **dictionary comprehensions**.

**Example:** A gradebook where each subject has more than one attribute (marks, credits, and faculty) — a nested dictionary.

```python
gradebook = {
    "DSA": {"marks": 92, "credits": 4, "faculty": "Dr. Rao"},
    "OS": {"marks": 78, "credits": 3, "faculty": "Prof. Kumar"},
    "DBMS": {"marks": 85, "credits": 4, "faculty": "Dr. Sharma"}
}

print(gradebook["DSA"]["marks"])              # 92
print(gradebook["OS"]["faculty"])             # Prof. Kumar

# .get() avoids a crash if the key doesn't exist
print(gradebook.get("AI", "Subject not enrolled"))
```

**Dictionary comprehension** — building a simple `{subject: marks}` view from the nested gradebook in one line:

```python
marks_only = {subject: details["marks"] for subject, details in gradebook.items()}
print(marks_only)   # {'DSA': 92, 'OS': 78, 'DBMS': 85}
```

---

## Putting All Four Together — A Semester Report

```python
gradebook = {
    "DSA": {"marks": 92, "credits": 4},
    "OS": {"marks": 78, "credits": 3},
    "DBMS": {"marks": 85, "credits": 4},
    "CN": {"marks": 88, "credits": 3}
}

lab_submissions = ["DSA", "OS", "DSA", "DBMS", "CN", "OS", "OS"]   # LIST: every submission, in order

unique_subjects_submitted = set(lab_submissions)          # SET: which subjects had labs this week, no repeats

high_scoring_subjects = {s for s in unique_subjects_submitted if gradebook[s]["marks"] >= 85}   # set comprehension

best_subject_name = max(gradebook, key=lambda s: gradebook[s]["marks"])
best_subject = (best_subject_name, gradebook[best_subject_name]["marks"])   # TUPLE: fixed (subject, marks) record

print("All lab submissions:", lab_submissions)
print("Unique subjects submitted:", unique_subjects_submitted)
print("High-scoring subjects (>=85):", high_scoring_subjects)
print(f"Best subject: {best_subject[0]} ({best_subject[1]} marks)")
```

This mirrors a real semester report: the **list** is the raw submission log, the **set** answers "which subjects had activity" without duplicates, the **dictionary** answers "what are this subject's marks/credits", and the **tuple** locks in a fixed "best subject" fact that shouldn't be edited afterward.

---

## Quick Reference — Which One Do I Reach For?

| Question | Use |
|---|---|
| Will this collection change (add/remove) during the program? | **List** |
| Is this a fixed, small record that must never be edited? | **Tuple** |
| Do I need to remove duplicates or compare two collections? | **Set** |
| Do I need to look something up by name/key? | **Dictionary** |

Practice exercises for this material are in **Lab2_DataStructures_Practice.ipynb**.

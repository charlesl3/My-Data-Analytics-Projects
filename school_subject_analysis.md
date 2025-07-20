
# 🏫 School Subject Coverage Analysis (Pandas Pipeline)

This notebook-style workflow analyzes subject coverage across schools and states based on a raw dataset where subjects are space-separated strings. The goal is to clean and transform the data to show how many schools in each state offer key subjects.

---

## 📄 Problem Summary

We are given a dataset with the following structure:

| school_id | school_name | state_code | subject_name             |
|-----------|-------------|------------|---------------------------|
| 1         | A High      | NY-1       | Math English Chemistry    |
| 2         | B High      | CA@        | Math History              |
| 3         | C High      | TX#        | Math Chemistry Physics    |

### ❗ Notes:
- `subject_name` contains space-separated subject strings.
- `state_code` may contain non-alphanumeric characters (e.g. `NY-1`, `CA@`, `TX#`).
- Some schools may offer fewer than 3 subjects.

---

## 🎯 Final Output

A summary table per `state_code`, showing:

| state_code | english_schools | math_schools | physics_schools | chemistry_schools |
|------------|------------------|--------------|------------------|--------------------|

Only schools offering **3 or more unique subjects** are included.

---

## 🧪 Step-by-Step in Pandas

### Step 0. Sample Data

```python
import pandas as pd

data = {
    'school_id': [1, 2, 3, 4, 5, 6],
    'school_name': ['A High', 'B High', 'C High', 'D High', 'E High', 'F High'],
    'state_code': ['NY-1', 'CA@', 'TX#', 'CA@', 'NY-1', 'FL*'],
    'subject_name': [
        'Math English Chemistry',
        'Math History',
        'Math Chemistry Physics',
        'English',
        'Math English',
        'Physics Chemistry English'
    ]
}

df = pd.DataFrame(data)
```

---

### Step 1. Split Subjects into Lists

```python
df['subject_name'] = df['subject_name'].str.split()
```

---

### Step 2. Explode Subjects to One Per Row

```python
df = df.explode('subject_name')
```

---

### Step 3. Clean `state_code`

```python
df['state_code'] = df['state_code'].str.replace(r'[^A-Za-z0-9]', '', regex=True)
```

---

### Step 4. Filter Schools with ≥ 3 Unique Subjects

```python
subject_counts = df.groupby('school_id')['subject_name'].nunique()
valid_school_ids = subject_counts[subject_counts >= 3].index
df = df[df['school_id'].isin(valid_school_ids)]
```

---

### Step 5. Filter for Core Subjects

```python
target_subjects = ['English', 'Math', 'Physics', 'Chemistry']
df_filtered = df[df['subject_name'].isin(target_subjects)]
```

---

### Step 6. Group by State and Subject, Count Unique Schools

```python
df_unique = df_filtered.drop_duplicates(subset=['school_id', 'subject_name'])

grouped = df_unique.groupby(['state_code', 'subject_name'])['school_id']     .nunique().unstack(fill_value=0)

# or
grouped = df_unique.groupby(['state_code', 'subject_name'])['school_id'].agg('count').reset_index()
grouped = grouped.pivot(index='state_code', columns='subject_name', values='school_id').fillna(0).astype(int)
```

---

### Step 7. Rename and Reorder Columns

```python
for subj in target_subjects:
    if subj not in grouped.columns:
        grouped[subj] = 0

grouped = grouped.rename(columns={
    'English': 'english_schools',
    'Math': 'math_schools',
    'Physics': 'physics_schools',
    'Chemistry': 'chemistry_schools'
})

grouped = grouped[['english_schools', 'math_schools', 'physics_schools', 'chemistry_schools']]
grouped = grouped.reset_index()
```

---

## ✅ Final Result

The resulting DataFrame `grouped` shows, for each cleaned `state_code`, how many schools offer each core subject — with only schools offering ≥ 3 subjects included.

---

## 📌 Concepts Practiced

- `.str.split()` and `.explode()` for multi-valued cells
- Regex cleaning with `.str.replace()`
- Filtering with `.groupby().nunique()`
- Pivoting with `.unstack()` vs `.pivot()`
- Defensive coding with missing subjects
 

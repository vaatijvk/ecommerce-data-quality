# 🧹 Ecommerce Data Quality Audit
### Automated Data Cleaning & Issue Detection with Python | Moringa School Capstone

---

## 📌 Project Overview

In the real world, data is almost never clean. Whether it comes from a POS system, a CRM export, a web scrape, or a manual spreadsheet — it arrives with missing values, duplicate records, formatting inconsistencies, outliers, and silent errors that are invisible to the naked eye but destructive to any downstream analysis.

This project builds an **automated data quality auditing system** in Python that scans any e-commerce CSV dataset and systematically detects, flags, and reports every category of data "dirt" — without manually inspecting rows one by one.

The audit notebook adds a dedicated boolean `FLAG_` column for every issue it finds, meaning you can filter to exactly the rows that are problematic, fix them, and re-run the audit to confirm they are resolved. At the end, the notebook computes an **overall data quality score** — the percentage of rows in the dataset that have zero flagged issues — giving a single, clear metric for how trustworthy the data is before it goes into a dashboard, report, or machine learning pipeline.

This project was built as a capstone at **Moringa School** using generative AI (Claude via ai.moringaschool.com) as a learning and debugging partner throughout the development process.

---

## 🎯 Why This Project Matters

Dirty data causes real, costly problems:

- A `groupby("product_category")` treats `"Electronics"` and `"electronics "` as two separate categories — silently splitting your sales figures
- A `price` column stored as a string (`"$49.99"`) crashes any arithmetic operation without warning
- Duplicate `order_id` values make revenue totals double-count transactions
- A single outlier with a price of `$99999` instead of `$99.99` (a typo) skews your entire average price calculation

Most of these issues produce **no error messages** — the code runs, the results look plausible, and the analysis is wrong. This notebook makes those problems visible, measurable, and fixable.

---

## ✅ What the Audit Detects

| Category | What is checked | How it is flagged |
|----------|----------------|-------------------|
| **Missing values** | Null/NaN cells per column | `FLAG_missing_{column}` |
| **Duplicate rows** | Fully identical rows | `FLAG_duplicate_row` |
| **Duplicate IDs** | Non-unique order/customer IDs | `FLAG_duplicate_{id_col}` |
| **Type inconsistencies** | Numbers stored as strings (mixed types) | `FLAG_mixed_type_{column}` |
| **Invalid dates** | Dates that cannot be parsed | `FLAG_invalid_date_{column}` |
| **Outliers** | Values beyond 1.5× IQR fence | `FLAG_outlier_{column}` |
| **Negative values** | Negative prices or quantities | `FLAG_negative_price`, `FLAG_negative_qty` |
| **Whitespace** | Leading/trailing spaces in text | `FLAG_whitespace_{column}` |
| **Casing issues** | Mixed case in categorical columns | Printed warning (context-dependent) |

Every flag column is a simple `True`/`False` per row, making it easy to filter, count, and visualise affected rows.

---

## 📁 Project Structure

```
ecommerce-data-quality/
│
├── notebooks/
│   └── data_quality_audit.ipynb    ← Main Jupyter Notebook (run this)
│
├── data/
│   └── .gitkeep                    ← Folder placeholder (raw data not committed)
│
├── outputs/
│   └── dirty_ecommerce_flagged.csv ← Dataset with all FLAG_ columns added
│
├── TOOLKIT.md                      ← Full capstone toolkit document
├── requirements.txt                ← Python dependencies
├── .gitignore                      ← Excludes raw data and system files
└── README.md                       ← This file
```

> **Note on data privacy:** Raw CSV files are excluded from this repository via `.gitignore`. Only the flagged output is committed.

---

## 🛠️ System Requirements

| Item | Requirement |
|------|-------------|
| Operating System | Windows 10/11, macOS, or Linux |
| Python version | 3.8 or higher |
| Editor | Jupyter Notebook or JupyterLab |
| Key packages | pandas, numpy, matplotlib, seaborn |

---

## ⚙️ Installation & Setup

### 1. Download this repository

Click the green **Code** button on this page → **Download ZIP** → extract the folder.

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Add your dataset

Place your CSV file in the `data/` folder, then open the notebook and update the file path in Cell 1:

```python
# Windows
df = pd.read_csv(r"C:\Users\JUDITH\Downloads\dirty_ecommerce_dataset.csv")

# Mac / Linux
df = pd.read_csv("~/Downloads/dirty_ecommerce_dataset.csv")
```

> ⚠️ **Windows users:** Always use `r"..."` (raw string) before your file path. Without it, Python misreads backslashes like `\U` and `\D` as Unicode escape sequences and throws a `SyntaxError`.

### 4. Launch Jupyter Notebook

```bash
jupyter notebook
```

Open `notebooks/data_quality_audit.ipynb` and run all cells top to bottom (**Kernel → Restart & Run All**).

---

## 📊 Expected Output

After running the full notebook you will see:

**1. A missing value heatmap** — a colour-coded grid showing exactly which cells are null across the dataset.

**2. Console output per check**, for example:
```
=== MISSING VALUES ===
               missing_count  missing_%
customer_email            87      17.40
ship_date                 34       6.80

=== DUPLICATES ===
Full duplicate rows: 12 (2.40%)
  Duplicate order_id: 18 rows

=== OUTLIERS (IQR method) ===
  'price': 23 outliers (range -15.25–312.50)
  'quantity': 8 outliers (range -1.00–14.00)

=== STRING ISSUES ===
  'product_category': possible casing inconsistency (14 raw vs 7 lowered unique values)
  'city': 31 values with leading/trailing spaces
```

**3. A data quality summary bar chart** — horizontal bars showing how many rows each flag type caught.

**4. A printed quality score:**
```
Overall quality score: 63.4% rows have no flagged issues
```

**5. A flagged CSV export** — the original dataset with all `FLAG_` columns appended, saved to `outputs/`.

---

## 🧠 How AI Was Used in This Project

This project was developed using **Claude (via ai.moringaschool.com)** as a learning and debugging tool throughout. The AI was used as a knowledgeable partner to explain concepts, diagnose errors, and suggest improvements — not to blindly generate code.

Key moments where AI accelerated the work:

- **Scaffolding the structure:** The initial prompt asking for a step-by-step data quality audit notebook produced a clear 7-cell structure that mapped directly to the final notebook, saving significant research time.
- **Diagnosing the Windows path error:** Pasting the exact `SyntaxError` into the AI immediately surfaced the raw string fix (`r"..."`), along with an explanation of *why* it happens — retained knowledge, not just a copy-paste fix.
- **Fixing syntax bugs:** When a cell had a stray `(` before a comment and a double `))` at the end of a print statement, the AI pinpointed all three bugs in one response with precise, line-level explanations.
- **Justifying design decisions:** Asking "is the string inconsistency check necessary?" led to a clear explanation of how whitespace and casing issues silently corrupt `groupby` and merge operations — deepening understanding beyond just making code run.

Full prompt journal with evaluations is documented in `TOOLKIT.md` Section 6.

---

## 🐛 Common Errors & Fixes

### `SyntaxError: (unicode error) 'unicodeescape' codec can't decode bytes`
**Cause:** Windows file path with backslashes interpreted as Python escape sequences.
**Fix:** Add `r` before the path string:
```python
df = pd.read_csv(r"C:\Users\JUDITH\Downloads\file.csv")
```

### `SyntaxError: invalid syntax` on a print statement
**Cause:** Extra closing `)` at the end of a `print()` call, or a stray `(` before a `#` comment.
**Fix:** Check the end of every `print(...)` line for `))` and every inline comment for an unmatched opening bracket.

### `KeyError` on a FLAG_ column name
**Cause:** A space accidentally typed inside the column name string, e.g. `"FLAG_negative_ qty"`.
**Fix:** Remove the space — `"FLAG_negative_qty"`.

### Outlier cell catches FLAG_ columns themselves
**Cause:** Boolean `FLAG_` columns are stored as 0/1 integers and picked up by `select_dtypes(include=[np.number])`.
**Fix:** Already handled by the filter `[c for c in numeric_cols if not c.startswith("FLAG_")]` — but only if cells are run in order from top to bottom.

---

## 📚 References

| Resource | Link |
|----------|------|
| pandas documentation | https://pandas.pydata.org/docs/ |
| seaborn heatmap | https://seaborn.pydata.org/generated/seaborn.heatmap.html |
| IQR outlier method explained | https://www.thoughtco.com/what-is-the-interquartile-range-rule-3126244 |
| Python raw strings | https://docs.python.org/3/reference/lexical_analysis.html#string-and-bytes-literals |
| Jupyter Notebook quickstart | https://jupyter-notebook.readthedocs.io/en/stable/notebook.html |

---

## 👩‍💻 Author

**Judith** — Moringa School Data Science Track

*Built with Python, pandas, and a healthy suspicion of "clean-looking" data.*

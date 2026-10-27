# Lab Orders Data Cleaning

A Jupyter notebook that cleans a messy lab-orders table using **pandas** and a little **regex**.
It reads the raw CSV, fixes the problems, and saves a clean CSV.

---

## Files

| File | What it is |
|---|---|
| `data_cleaning_easy.ipynb` | The notebook (step-by-step, with explanations) |
| `lab_orders_raw.csv` | Input: the original messy data |
| `lab_orders_clean.csv` | Output: the cleaned data (created when you run the notebook) |

---

## Requirements

- Python 3.9 or newer
- pandas 2.0 or newer
- numpy
- Jupyter Notebook or JupyterLab

Install:

```bash
pip install pandas numpy notebook
```

---

## How to run

1. Put `lab_orders_raw.csv` in the **same folder** as the notebook.
2. Start Jupyter:
   ```bash
   jupyter notebook
   ```
3. Open `data_cleaning_easy.ipynb`.
4. Run all cells from top to bottom (`Shift + Enter`, or menu: *Run > Run All Cells*).
5. Find `lab_orders_clean.csv` in the same folder.

---

## What the notebook does

| Step | Problem | Fix |
|---|---|---|
| 1 | Data types guessed wrongly | Read every column as text first |
| 2 | Hidden spelling differences | Use `value_counts()` to find them |
| 3 | `Unknown`, `NaN`, `NS`, `invalid`, empty cells | Convert them to real missing values (`NaN`) |
| 4 | 3 different date styles | `pd.to_datetime(..., format="mixed")` |
| 5 | Prices like `ksh 3833` | Regex `[^0-9]` removes everything except digits |
| 6 | `Haematology` / `Hematology`, `IHC`, `lab a`, `INS` ... | Dictionary mapping + Title Case |
| 7 | `Dr.`, `Dr`, or no title on doctor names | Regex removes the title, then `Dr.` is added to everyone |
| 8 | Repeated order IDs | Flagged in a new column, not deleted |
| 9 | - | Save to `lab_orders_clean.csv` |

### Regex patterns used

| Pattern | Meaning | Used for |
|---|---|---|
| `(?i)^(unknown\|nan\|ns\|not specified\|invalid\|)$` | Whole cell is one of these words (any capitals) or empty | Missing values |
| `[^0-9]` | Anything that is not a digit | Prices |
| `(?i)^st\.? mary.*` | Starts with "St", optional dot, then "Mary" | Facility names |
| `(?i)^dr\.?\s*` | "Dr" at the start, optional dot, spaces | Doctor names |

---

## Assumptions (please check these)

1. **Dates like `11/09/2022` are month/day/year.** This matches the clear examples such as `03/16/2021`.
2. **All prices are in KSh**, even where the `ksh` text is missing.
3. **A price of `0` means "not entered"**, so it is turned into a missing value.
4. **`Dr. Smyth` is a typo of `Dr. Smith`.**
5. **`St. Marys` and `St. Mary's Oncology` are the same facility.**

If any of these is wrong for your data, change the matching line in the notebook.

---

## Output columns

The clean file has the same columns as the input, plus:

| Column | Meaning |
|---|---|
| `order_id_repeated` | `True` if the same `order_id` appears on more than one row (different patients, so likely an ID mistake: review these) |

Cleaned columns:

| Column | Type | Notes |
|---|---|---|
| `signout_date`, `creation_date` | date | `YYYY-MM-DD` |
| `price` | number | Digits only, `0` becomes missing |
| `test_category`, `sub_category`, `payment_method` | text | One standard spelling each |
| `facility`, `receiving_centers`, `processing_centers`, `service` | text | Title Case |
| `assigned_to_pathologist` | text | Format `Dr. First Last` |

---

## Known limitations

- Many rows have `creation_date` **after** `signout_date`, which looks wrong. The notebook does not change this; check the source system.
- About a third of `sub_category`, `payment_method` and `processing_centers` values are missing in the raw data, so they stay missing.
- Date rows that cannot be read become missing instead of stopping the script (`errors="coerce"`).

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `FileNotFoundError: lab_orders_raw.csv` | CSV is in a different folder | Move the CSV next to the notebook |
| `ValueError` on `format="mixed"` | pandas older than 2.0 | `pip install --upgrade pandas` |
| `ModuleNotFoundError: pandas` | pandas not installed | `pip install pandas` |

---

## How to reuse on new data

1. Replace `lab_orders_raw.csv` with the new file (same column names).
2. Run all cells.
3. Look at the `value_counts()` output in Step 2 and Step 6. New spelling variants will show up there. Add them to the dictionaries in Step 6.

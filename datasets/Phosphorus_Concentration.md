## Phosphorus-Concentration-tser

### 1. What the data is about

One record is one soil sample.

Each record is a univariate equal-length series of 3578 floats plus a scalar target `y` (`@problemName PhosphorusConcentration`, `@univariate true`, `@equalLength true`, `@seriesLength 3578`, `@targetlabel true`). The dump README states the series is “a spectrum treated as a univariate series” and `y` is “a lab-measured elemental concentration”.

2248 samples (1573 train / 675 test)

**Missingness:** 0 NaN and 0 Inf in `X_train`, `y_train`, `X_test`, `y_test`. TRAIN/TEST headers have no `@missing` tag. Every `.ts` data row has exactly one `:` (one channel + target). Row counts match the npz first axis (1573 / 675).

---

### 2. Task input and output

- **Task:** single-modal sequential regression — predict one continuous phosphorus concentration from one univariate vis-NIR spectrum.
- **Input**
  - **Sequential:** one channel, length `T = 3578`, `dtype float32`, per-sample shape `(1, 3578)` (npz batch shape `(n, 1, 3578)`). `.ts` stores the channel as comma-separated floats; npz keys `X_train` / `X_test`. `@timestamps false`. High-cardinality on all 5,628,194 train values: min −0.06769, max 2.544, 5,008,571 distinct, mode 0.26647 (count 8). 5,533 train values are negative (0.098% of cells). No table or image modality.
- **Output / target:** one `float64` scalar `y` (npz `y_train` / `y_test`). Observed on the 1573 train rows (primary split):

    | target | min | median | max | skew |
    |---|---|---|---|---|
    | `y` | 0.001 | 5.162 | 396.0 | 6.84 |

    877 distinct train values; mode `1.53` (9 rows). `y = 0.001` on 2 train rows (the observed minimum). Test (675 rows): min 0.001, median 5.324, max 382.7, skew 9.12.
- **Auxiliary:** `split` (`train` / `test`, from `PhosphorusConcentration_TRAIN.ts` / `_TEST.ts`). No sample-id field in the `.ts` or npz.

---

### 3. Case study

```json
{
  "split": "train",
  "index": 0,
  "y": 3.238
}
```

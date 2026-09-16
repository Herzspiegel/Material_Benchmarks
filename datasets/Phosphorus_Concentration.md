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

Train index 0. Sequential input `X` is `float32` shape `(1, 3578)`; `head` is `X[0, 0:32]`, `tail` is `X[0, -8:]`.

```json
{
  "split": "train",
  "index": 0,
  "X": {
    "dtype": "float32",
    "shape": [1, 3578],
    "head": [
      1.6502553224563599,
      1.655259370803833,
      1.659000039100647,
      1.6647971868515015,
      1.6705667972564697,
      1.6777740716934204,
      1.687721848487854,
      1.6963917016983032,
      1.6978429555892944,
      1.695381999015808,
      1.6963917016983032,
      1.7046518325805664,
      1.7203766107559204,
      1.7342618703842163,
      1.74186372756958,
      1.7471333742141724,
      1.7428021430969238,
      1.7253552675247192,
      1.7092334032058716,
      1.7041966915130615,
      1.7034646272659302,
      1.6991243362426758,
      1.6933200359344482,
      1.6898491382598877,
      1.6895105838775635,
      1.6925128698349,
      1.6996073722839355,
      1.7128835916519165,
      1.728867769241333,
      1.7387126684188843,
      1.7428040504455566,
      1.748213291168213
    ],
    "tail": [
      0.2605981230735779,
      0.2634321451187134,
      0.26584896445274353,
      0.2649171054363251,
      0.26269614696502686,
      0.2620617151260376,
      0.2631704807281494,
      0.26422804594039917
    ],
    "min": 0.24936001002788544,
    "max": 1.859097957611084,
    "mean": 0.6828802824020386
  },
  "y": 3.238
}
```


# Nanofiber Composite (ORDER / MatMCL) — `io_nn_ready`

Material data benchmark of the **preprocessed** dump (one ID = one sample; direction already pivoted). Produced with the [dataset-brief](../../docs/dataset-brief-workflow.md) workflow. Evidence is the files on `10.120.17.1`.

| | |
|---|---|
| **Catalog id** | `nanofiber-composite-order-io-nn-ready` |
| **Path inspected** | `/sde/shz/data/Nanofiber_ORDER/io_nn_ready` |
| **Primary table** | `id_index/table.csv` (235 rows) |
| **Parallel layout** | `jsonl/records.jsonl` (235 objects) |
| **Inspected** | 2026-09-11 |

`inventory.docs` was empty. `manifest.json` is treated as **claims**. CSV / JSONL / JPEG pixels are **observation**. Where they disagree, observation wins.

---

### 1. What the data is about

One record is one electrospun nanofiber membrane, keyed by integer `ID`. After this project’s merge/pivot, that ID has six process settings, a bag of SEM crops, and ten tensile numbers (five properties × two test directions). There is no `dir` column left to join.

**Population (from the dump):** 235 labeled IDs, min 2 / max 240. IDs missing in 2–240: `{41, 181, 224, 225}`. `manifest.json` also lists image-only ID `1` as dropped: *“IDs with images but no mechanical table were dropped.”* `excluded_image_only_ids = [1, 41, 181, 224, 225]`. Source image tree `matmcl_data/datasets/images/preprocessed` has 240 ID folders; the dump has 235; set difference is exactly those five.

**How the dump was built (claim, `manifest.json` + `build_io_nn_ready.py`):** created `2026-09-03T08:11:31Z` by concatenating original `train.csv` / `val.csv` / `test.csv`, requiring exactly two `dir` ∈ {0,1} rows per ID with identical process features, pivoting five named targets into ten columns, keeping only IDs that also have an image folder, and **hard-linking** crops (not copying). Manifest `versions.*.root` still prints `/ai/data/shz/data/Nanofiber_ORDER/io_nn_ready`. On the inspected host the files sit under `/sde/shz/data/Nanofiber_ORDER/io_nn_ready`.

Two equivalent layouts of the same 235 samples:

| Layout | Table | Images |
|---|---|---|
| Integer-ID | `id_index/table.csv` | `id_index/images/preprocessed/<ID>/<ID>_<k>.jpg` |
| JSON / JSONL | `jsonl/records.jsonl` (and `records.json`, length 235) | `jsonl/images/preprocessed/<ID>/` |

Crops are hard-linked across the two trees and the source folder (`nlink=3`, same inode). File walk counts 33,000 `.jpg` entries because both trees are listed; unique crops = **16,500**. Per-ID bag size 30 / 60 / 150 (min / median / max). Histogram: 30→2, 45→1, 60→134, 75→60, 90→21, 105→9, 120→5, 135→2, 150→1. Every JSONL `n_crops` matches `len(images)` and the folder JPEG count (0 mismatches / 235). Crop indices are `0..n_crops-1` with no gaps.

**Missingness:** 0 empty cells in all 17 CSV columns. Every table ID has an image folder. CSV ↔ JSONL max abs difference **0.0** on all 6 features + 10 targets, same row order (sorted by `ID`). Source mech CSVs match the dump with max abs **0** over 2,350 target cells.

**Splits:** none in this dump. Native ORDER files still exist next door and are ID-disjoint: train 164 / val 35 / test 36 IDs (328+70+72 = 470 direction-rows). The dump merges them on purpose. Published UQ runs used a different 120/21/47/47 ID permutation; that permutation is not stored here.

**Modeling notes from the files:**

- All 235 `(f,c,v,r,t,w)` vectors are unique. The 4-tuple `(f,c,v,r)` is already unique (235 / 240 possible cells occupied once).
- `f,c,v,r` are nearly balanced grids. `t` is not: **108 / 235** rows have `t=20`; `t=28` has 1 row. `w` takes 52 distinct values.
- Closest pairs differ by one step of `f` only (e.g. ID 86 vs 166). An ID-random split can put recipe twins on opposite sides of train/test.
- Target scales differ by orders of magnitude (elongation ~20–187 vs moduli ~0.01–1.4). dir0/dir1 are **not** copies of one property (fracture corr −0.12; elastic modulus 0.07; tangent 0.00). Collector speed `r` tracks anisotropy: mean E_dir1/E_dir0 goes from 0.80 at `r=0` to 4.02 at `r=1600`.
- JPEG filenames are `<ID>_<k>.jpg` (identity in the path). `sorted(glob("*.jpg"))` is lexical (`2_10` before `2_2`); JSONL lists numeric order.
- RGB JPEGs are grayscale triples (`gray_frac=1.0` on inspected files).
- Column names `f,c,v,r,t,w` are unexplained in the dump. Project brief `IO/Data_Scientist.md` claims: flow rate, solution concentration, spinning voltage, collector rotation, ambient temperature, ambient humidity. Units are not in the dump; that file claims strengths/moduli on a tensile-stress scale (MPa) and elongation in %.

**Claim vs observation:** `ORDER/README.md` says *“~200 samples. 7 features including categorical fiber direction, 5 targets.”* Files have **235** labeled IDs; “7 features” is 6 process columns + `dir`. Trust 235.

This dump is only the **mechanical-property** task. Sibling folders `table/gen`, `table/sgpt`, `table/retrieval_gallery`, and `images/gen` (15,645 flat JPEGs) are other ORDER tasks and are not in `io_nn_ready`.

---

### 2. Task input and output

- **Task:** multimodal multi-output regression — predict ten continuous tensile properties of one membrane from process settings + an SEM crop bag.
- **Input**
  - **Table:** 6 scalars. Observed dtypes and ranges on all 235 rows:

    | field | dtype | observed values | counts |
    |---|---|---|---|
    | `f` | float | `{0.2, 0.4, 0.6}` | 78 / 80 / 77 |
    | `c` | int | `{16, 18, 20, 22}` | 59 / 59 / 59 / 58 |
    | `v` | int | `{16, 18, 20, 22}` | 55 / 60 / 60 / 60 |
    | `r` | int | `{0, 400, 800, 1200, 1600}` | 45 / 48 / 48 / 47 / 47 |
    | `t` | int | `{18,…,28}` | 108 rows at 20; tail sparse |
    | `w` | int | 13–82 (52 distinct) | mode `w=50` with 21 rows |

    Missingness: none. CSV stores integers without `.0` (`2,0.2,16,16,400,20,48,…`).
  - **Images:** unordered bag of RGB JPEG crops, `H×W×3 = 512×512×3` (PIL: size `(512, 512)`, mode `RGB`, format `JPEG`; R=G=B). `K ∈ [30, 150]` per ID. JSONL paths are relative to `jsonl/`, e.g. `images/preprocessed/2/2_0.jpg`. Survey: 435 opened JPEGs (every crop 0 + 200 random others) were all 512×512 RGB.
- **Output / target:** 10 floats, already pivoted. CSV is flat `*_dir0` / `*_dir1`. JSONL is nested `targets.<property>.dir0|dir1`. Ranges on 235 rows:

    | target | min | median | max | skew |
    |---|---|---|---|---|
    | `fracture_dir0` | 1.017 | 7.729 | 16.950 | 0.28 |
    | `fracture_dir1` | 3.187 | 9.966 | 30.531 | 1.25 |
    | `elongation_dir0` | 29.191 | 99.771 | 186.853 | 0.05 |
    | `elongation_dir1` | 22.113 | 75.649 | 176.298 | 0.43 |
    | `elastic_modulus_dir0` | 0.032 | 0.246 | 0.950 | 1.53 |
    | `elastic_modulus_dir1` | 0.104 | 0.375 | 1.391 | 1.22 |
    | `tangent_modulus_dir0` | 0.010 | 0.067 | 0.367 | 2.53 |
    | `tangent_modulus_dir1` | 0.024 | 0.106 | 0.929 | 2.25 |
    | `yield_dir0` | 0.340 | 2.901 | 9.993 | 1.43 |
    | `yield_dir1` | 0.984 | 4.208 | 28.110 | 2.64 |

    dir1 is systematically stronger / stiffer / less extensible except at `r=0` (mean fracture dir1/dir0 = 0.85 at `r=0`, 2.61 at `r=1600`).
- **Auxiliary:** `ID` / `id` (int, sample key); `n_crops` (int, equals bag size); `images` (`array<string>` of relative JPEG paths). No split flag in the dump.

JSONL schema (inspect of `records.jsonl`):

```
{
  id: int,
  features: {f: float, c: int, v: int, r: int, t: int, w: int},
  targets: {
    fracture: {dir0: float, dir1: float},
    elongation: {dir0: float, dir1: float},
    elastic_modulus: {dir0: float, dir1: float},
    tangent_modulus: {dir0: float, dir1: float},
    yield: {dir0: float, dir1: float}
  },
  images: array<string>,
  n_crops: int
}
```

CSV schema: `ID:int` + six features + ten `*_dir0/1` floats. 235 rows, exact.

---

### 3. Case study

- **Locator:** `/sde/shz/data/Nanofiber_ORDER/io_nn_ready/id_index/table.csv`, index 0 (same membrane as JSONL line 0, `id=2`). Native ORDER split of this ID: `train`.
- **Replay:** `python3 inspect_dataset.py /sde/shz/data/Nanofiber_ORDER/io_nn_ready --index 0`  
  Nested form: `python3 inspect_dataset.py /sde/shz/data/Nanofiber_ORDER/io_nn_ready/jsonl/records.jsonl --index 0`
- **Raw (CSV sample, untruncated):**

```json
{
  "ID": 2,
  "f": 0.2,
  "c": 16,
  "v": 16,
  "r": 400,
  "t": 20,
  "w": 48,
  "fracture_dir0": 10.84147493,
  "fracture_dir1": 9.563418679,
  "elongation_dir0": 42.41121928,
  "elongation_dir1": 49.81189092,
  "elastic_modulus_dir0": 0.496056063,
  "elastic_modulus_dir1": 0.374883192,
  "tangent_modulus_dir0": 0.231847925,
  "tangent_modulus_dir1": 0.174534157,
  "yield_dir0": 5.688269252,
  "yield_dir1": 4.372749039
}
```

JSONL `images` truncated by the inspector to 12 of 60 paths (`truncated_fields: ["images"]`): `images/preprocessed/2/2_0.jpg` … `2_11.jpg`, `_len: 60`. `n_crops: 60`.

- **Walkthrough**
  - `ID=2` — membrane key; folder `.../preprocessed/2/` is the bag.
  - `f,c,v,r,t,w = 0.2, 16, 16, 400, 20, 48` — table input `X` (low flow / concentration / voltage, collector 400).
  - Ten `*_dir0/1` values — target `y`. Relatively stiff in dir0 (`elastic_modulus_dir0=0.496` vs dump median 0.246; `fracture_dir0=10.84` vs median 7.73) and relatively inextensible (`elongation_dir0=42.4` vs median 99.8).
  - `images` / `n_crops=60` — image input. Inspected file: `id_index/images/preprocessed/2/2_0.jpg` (hard-linked to the jsonl copy and to `matmcl_data/.../2/2_0.jpg`). **Pixels:** square SEM of a nonwoven fiber mat — overlapping randomly oriented filaments, a few beads, bright fibers on a dark porous background. Encoded as RGB 512×512 with identical channels (mean intensity ≈ 101). Crops 30 and 59 of the same bag are the same morphology, not the same tile (crop 59 has a debris/bead blob). Bag pooling is not averaging identical patches.

Other records opened for the brief (not the skill default sample): ID 86 (`r=0`, isotropic mat, fracture 10.36/9.43); ID 199 (builder self-check, CSV index 195, `n_crops=90`, features `{0.6,18,22,1200,20,53}`); ID 215 (CSV index 211, `r=1600`, strongly aligned thicker fibers, fracture 6.38/**30.53**). Replay ID 199: `--index 195`. Replay ID 215: `--index 211`.

---

### Limits

- No README / dataset card inside `io_nn_ready`. Feature glossary and physical units are not stored next to the numbers.
- JPEG geometry checked on 435 / 16,500 files (all 512×512 RGB in that sample). Within-bag MD5 uniqueness checked on the first 40 IDs only; crop-0 MD5 is unique for all 235 IDs.
- Figshare was not re-downloaded. Wu et al. citation is from `ORDER/README.md`, not from a file inside the dump.
- `uq_cache/` tensors and `datasets.zip` were not opened. This brief is the `io_nn_ready` dump, not the full `Nanofiber_ORDER` tree.
- Native-split disjointness is on IDs, not on near-duplicate recipes.

---

## Benchmark notes (after the required brief)

These are extra observations that matter for using the dump as a test set. They are not a substitute for sections 1–3.

**Loader contract**

1. One row = one ID. Do not explode dir0/dir1 back into two samples unless you also duplicate the image bag.
2. Read images from the JSONL `images` list (numeric). Do not `sorted(glob("*.jpg"))`.
3. JSONL paths are relative to `jsonl/`, not to `io_nn_ready/`.
4. `n_crops` varies; pad or subsample. Mean K is 67–74 in every `r`/`c`/`f` bucket — bag size is not a process feature.
5. RGB JPEGs are grayscale triples; ImageNet-mean normalization is a color prior this data does not have.
6. Native `train.csv` / `val.csv` / `test.csv` are the **pre-pivot** 470-row tables. Reloading them as 328 “train samples” double-counts membranes.
7. Strip filenames before the model (`<ID>_<k>.jpg` leaks the key).

**Source**

- ORDER repo on disk: `/sde/shz/data/Nanofiber_ORDER/ORDER/` (MIT code license, Copyright 2026 Xinyao Li). Data pointer in that README: Figshare `https://figshare.com/s/0cad763a26f928b70840`, Wu et al., npj Computational Materials 11:276 (2025).
- Builder self-check still holds on this host: ID 199 has `n_crops=90` and features `{f:0.6, c:18, v:22, r:1200, t:20, w:53}`.

**What not to mix in**

Image-only IDs 1, 41, 181, 224, 225 still have JPEG folders in `matmcl_data` (60/60/105/90/60 crops) and are unlabeled. `images/gen/` uses the same `<ID>_<k>.jpg` names at the top level — generation gallery, not the mech bags.

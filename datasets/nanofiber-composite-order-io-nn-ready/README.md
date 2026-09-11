### 1. What the data is about

One record is one electrospun nanofiber membrane, keyed by integer `ID`.

After preprocessing, each ID has six process settings, a bag of SEM crops, and ten tensile numbers (five properties × two test directions).

235 labeled IDs

**Missingness:** 0 empty cells in all 17 CSV columns. Every table ID has an image folder.

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
- **Auxiliary:** `ID` / `id` (int, sample key); `n_crops` (int, equals bag size); `images` (`array<string>` of relative JPEG paths).

---

### 3. Case study

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

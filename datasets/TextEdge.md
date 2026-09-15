## TextEdge-llm-prop-v1

### 1. What the data is about

One record is one crystalline solid, keyed by Materials Project `material_id`.

Each record has

- a chemical `formula`
- a pymatgen CIF string `cif_structure`
- a Robocrystallographer structural `description`
- `band_gap`, unit-cell `volume`, and a boolean `is_gap_direct`

146574 crystals across three split CSVs (`train.csv` 125098 / `validation.csv` 9945 / `test.csv` 11531). 144862 distinct `material_id` values (146570 `mp-*`, 4 `mvc-*`).

**Missingness:** 0 empty cells in `material_id`, `formula`, `description`, and `band_gap`. 40 rows have null `volume` and null `is_gap_direct` together (train 33 / validation 5 / test 2). 370 test rows have empty `cif_structure` (the other 146204 CIFs are non-empty). `material_id` is unique on train and validation. Test has 9888 distinct IDs among 11531 rows: 9835 IDs appear once and 53 IDs appear 32 times each (same `description`/`volume`/`band_gap`; 8 distinct CIF strings per ID). 69 IDs occur in both train and test; train–validation and validation–test overlaps are 0.

---

### 2. Task input and output

- **Task:** single-modal multi-task crystal property prediction — predict `band_gap` (regression), `volume` (regression), and `is_gap_direct` (binary classification) of one crystal from its Robocrystallographer `description`.
- **Input**
  - **Text:** one UTF-8 string `description` (Robocrystallographer prose with unicode subscripts, charges, Å / °). Length on all 146574 rows: 152–69186 characters (median 1332); 27–9825 words (median 224). Index 0 is 3254 characters / 599 words.
- **Output / target:** two floats and one boolean, observed on all 146574 rows (`volume` / `is_gap_direct` stats exclude the 40 nulls where noted):

    | field | dtype | observed values | counts |
    |---|---|---|---|
    | `is_gap_direct` | bool | `{False, True, null}` | 125544 / 20990 / 40 |

    CSV stores `True` / `False` (train index 0 is `False`). Continuous targets:

    | target | min | median | max | skew |
    |---|---|---|---|---|
    | `band_gap` | 0.0 | 0.023 | 17.891 | 1.56 |
    | `volume` | 5.610 | 320.687 | 18368.448 | 4.84 |

    `band_gap` is 0.0 on 71582 rows (mode). `volume` has 40 nulls; 144582 distinct values among 146534 non-null rows.
- **Auxiliary:** `material_id` (string, sample key, 144862 distinct); `formula` (string, 99807 distinct, length 1–25 characters, median 8). `cif_structure` is a UTF-8 CIF text dump: 146204 non-empty strings start with `# generated using pymatgen` and set `_symmetry_space_group_name_H-M` to `'P 1'` (length 689–23775 characters, median 1693 on the pool including 370 empties). Split files are `train.csv`, `validation.csv`, and `test.csv`; there is no dataset README in the dump.

---

### 3. Case study

```json
{
  "material_id": "mp-759684",
  "formula": "Na3Bi(P2O7)2",
  "cif_structure": "# generated using pymatgen\ndata_Na3Bi(P2O7)2\n_symmetry_space_group_name_H-M   'P 1'\n_cell_length_a   7.40719600\n_cell_length_b   9.65342044\n_cell_length_c   9.71354802\n_cell_angle_alpha   80.67687532\n_cell_angle_beta   77.32513795\n_cell_angle_gamma   69.81272828\n_symmetry_Int_Tables_number   1\n_chemical_formula_structural   Na3Bi(P2O7)2\n_chemical_formula_sum   'Na6 Bi2 P8 O28'\n_cell_volume   633.18183015\n_cell_formula_units_Z   2\nloop_\n _symmetry_equiv_pos_site_id\n _symmetry_equiv_pos_as_xyz\n  1  'x, y, z'\nloop_\n _atom_site_type_symbol\n _atom_site_label\n _atom_site_symmetry_multiplicity\n _atom_site_fract_x\n _atom_site_fract_y\n _atom_site_fract_z\n _atom_site_occupancy\n  Na  Na0  1  0.35480800  0.14859700  0.27365400  1\n  Na  Na1  1  0.05244500  0.50077500  0.16174000  1\n  Na  Na2  1  0.44253400  0.45859900  0.33184400  1\n  Na  Na3  1  0.55746600  0.54140100  0.66815600  1\n  Na  Na4  1  0.94755500  0.49922500  0.83826000  1\n  Na  Na5  1  0.64519200  0.85140300  0.72634600  1\n  Bi  Bi6  1  0.88049800  0.12228900  0.78149300  1\n  Bi  Bi7  1  0.11950200  0.87771100  0.21850700  1\n  P  P8  1  0.18494400  0.13541900  0.97454300  1\n  P  P9  1  0.72862200  0.10770600  0.45107900  1\n  P  P10  1  0.41375300  0.33375700  0.94470100  1\n  P  P11  1  0.90209900  0.34153200  0.45273200  1\n  P  P12  1  0.09790100  0.65846800  0.54726800  1\n  P  P13  1  0.58624700  0.66624300  0.05529900  1\n  P  P14  1  0.27137800  0.89229400  0.54892100  1\n  P  P15  1  0.81505600  0.86458100  0.02545700  1\n  O  O16  1  0.17896500  0.03494500  0.62393000  1\n  O  O17  1  0.12234800  0.16301200  0.13073200  1\n  O  O18  1  0.76856700  0.02847100  0.04386200  1\n  O  O19  1  0.69150100  0.07786900  0.61431300  1\n  O  O20  1  0.03650500  0.23659200  0.87881100  1\n  O  O21  1  0.54446100  0.20865900  0.39712500  1\n  O  O22  1  0.38316000  0.18115400  0.91522000  1\n  O  O23  1  0.09477300  0.35653800  0.37337900  1\n  O  O24  1  0.41022600  0.33422800  0.10263800  1\n  O  O25  1  0.89620200  0.18656900  0.41132600  1\n  O  O26  1  0.60818300  0.32988900  0.84733800  1\n  O  O27  1  0.88252800  0.32474700  0.61647600  1\n  O  O28  1  0.24187900  0.46737100  0.90443700  1\n  O  O29  1  0.27797200  0.53260600  0.58560600  1\n  O  O30  1  0.72202800  0.46739400  0.41439400  1\n  O  O31  1  0.75812100  0.53262900  0.09556300  1\n  O  O32  1  0.11747200  0.67525300  0.38352400  1\n  O  O33  1  0.39181700  0.67011100  0.15266200  1\n  O  O34  1  0.10379800  0.81343100  0.58867400  1\n  O  O35  1  0.58977400  0.66577200  0.89736200  1\n  O  O36  1  0.90522700  0.64346200  0.62662100  1\n  O  O37  1  0.61684000  0.81884600  0.08478000  1\n  O  O38  1  0.45553900  0.79134100  0.60287500  1\n  O  O39  1  0.96349500  0.76340800  0.12118900  1\n  O  O40  1  0.30849900  0.92213100  0.38568700  1\n  O  O41  1  0.23143300  0.97152900  0.95613800  1\n  O  O42  1  0.87765200  0.83698800  0.86926800  1\n  O  O43  1  0.82103500  0.96505500  0.37607000  1\n",
  "description": "Na₃Bi(P₂O₇)₂ crystallizes in the triclinic P̅1 space group. There are three inequivalent Na¹⁺ sites. In the first Na¹⁺ site, Na¹⁺ is bonded to five O²⁻ atoms to form NaO₅ square pyramids that share corners with five PO₄ tetrahedra. There are a spread of Na–O bond distances ranging from 2.30–2.41 Å. In the second Na¹⁺ site, Na¹⁺ is bonded in a 5-coordinate geometry to five O²⁻ atoms. There are a spread of Na–O bond distances ranging from 2.31–2.59 Å. In the third Na¹⁺ site, Na¹⁺ is bonded in a 8-coordinate geometry to eight O²⁻ atoms. There are a spread of Na–O bond distances ranging from 2.29–3.08 Å. Bi⁵⁺ is bonded in a 7-coordinate geometry to seven O²⁻ atoms. There are a spread of Bi–O bond distances ranging from 2.27–2.75 Å. There are four inequivalent P⁵⁺ sites. In the first P⁵⁺ site, P⁵⁺ is bonded to four O²⁻ atoms to form PO₄ tetrahedra that share  a cornercorner with one NaO₅ square pyramid and  a cornercorner with one PO₄ tetrahedra. There are a spread of P–O bond distances ranging from 1.52–1.64 Å. In the second P⁵⁺ site, P⁵⁺ is bonded to four O²⁻ atoms to form PO₄ tetrahedra that share corners with two equivalent NaO₅ square pyramids and  a cornercorner with one PO₄ tetrahedra. There are a spread of P–O bond distances ranging from 1.52–1.62 Å. In the third P⁵⁺ site, P⁵⁺ is bonded to four O²⁻ atoms to form PO₄ tetrahedra that share  a cornercorner with one NaO₅ square pyramid and  a cornercorner with one PO₄ tetrahedra. There are a spread of P–O bond distances ranging from 1.53–1.64 Å. In the fourth P⁵⁺ site, P⁵⁺ is bonded to four O²⁻ atoms to form PO₄ tetrahedra that share  a cornercorner with one NaO₅ square pyramid and  a cornercorner with one PO₄ tetrahedra. There are a spread of P–O bond distances ranging from 1.51–1.63 Å. There are fourteen inequivalent O²⁻ sites. In the first O²⁻ site, O²⁻ is bonded in a distorted single-bond geometry to one Bi⁵⁺ and one P⁵⁺ atom. In the second O²⁻ site, O²⁻ is bonded in a 2-coordinate geometry to one Na¹⁺, one Bi⁵⁺, and one P⁵⁺ atom. In the third O²⁻ site, O²⁻ is bonded in a distorted single-bond geometry to one Bi⁵⁺ and one P⁵⁺ atom. In the fourth O²⁻ site, O²⁻ is bonded in a 3-coordinate geometry to one Na¹⁺, one Bi⁵⁺, and one P⁵⁺ atom. In the fifth O²⁻ site, O²⁻ is bonded in a distorted trigonal planar geometry to one Na¹⁺, one Bi⁵⁺, and one P⁵⁺ atom. In the sixth O²⁻ site, O²⁻ is bonded in a distorted trigonal planar geometry to two Na¹⁺ and one P⁵⁺ atom. In the seventh O²⁻ site, O²⁻ is bonded in a bent 120 degrees geometry to two P⁵⁺ atoms. In the eighth O²⁻ site, O²⁻ is bonded in a 3-coordinate geometry to three Na¹⁺ and one P⁵⁺ atom. In the ninth O²⁻ site, O²⁻ is bonded in a 4-coordinate geometry to three Na¹⁺ and one P⁵⁺ atom. In the tenth O²⁻ site, O²⁻ is bonded in a bent 120 degrees geometry to two P⁵⁺ atoms. In the eleventh O²⁻ site, O²⁻ is bonded in a 3-coordinate geometry to one Na¹⁺, one Bi⁵⁺, and one P⁵⁺ atom. In the twelfth O²⁻ site, O²⁻ is bonded in a distorted single-bond geometry to one Na¹⁺, one Bi⁵⁺, and one P⁵⁺ atom. In the thirteenth O²⁻ site, O²⁻ is bonded in a 3-coordinate geometry to three Na¹⁺ and one P⁵⁺ atom. In the fourteenth O²⁻ site, O²⁻ is bonded in a 3-coordinate geometry to two equivalent Na¹⁺ and one P⁵⁺ atom.",
  "band_gap": 0.0,
  "volume": 633.1818301532061,
  "is_gap_direct": false
}
```

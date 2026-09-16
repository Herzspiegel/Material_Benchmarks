# Material Benchmarks for Data Scientists

A catalog of **materials datasets** used to test prediction and uncertainty-quantification methods.

## Material data benchmarks

### Single-modal

**Text**

| ID | n | Dataset | Modalities | Task | Dump | Brief | Original Link |
|---|---|---|---|---|---|---|---|
| `textedge-llm-prop-v1` | 146,574 crystals | TextEdge (LLM-Prop) | Robocrystallographer text | band-gap / volume / is-gap-direct | `TextEdge_v.1` | [link](datasets/TextEdge.md) | [link](https://www.nature.com/articles/s41524-025-01536-2) |

**Image**

**Sequential**

| ID | n | Dataset | Modalities | Task | Dump | Brief | Original Link |
|---|---|---|---|---|---|---|---|
| `phosphorus-concentration-tser` | 2,248 samples | Phosphorus Concentration (TSER) | univariate vis-NIR series | scalar P-concentration regression | `TSER_Soil` | [link](datasets/Phosphorus_Concentration.md) | [link](https://zenodo.org/records/11236716) |

**Computational**

### Multi-modal

| ID | n | Dataset | Modalities | Task | Dump | Brief | Original Link |
|---|---|---|---|---|---|---|---|
| `nanofiber-composite-order-io-nn-ready` | 235 membranes | Nanofiber Composite (ORDER / MatMCL) | table + SEM image bags | 10-way tensile regression | preprocessed `io_nn_ready` | [link](datasets/Nanofiber_Composite.md) | [link](https://www.nature.com/articles/s41524-025-01767-3) |
| `hmof-llm4mat-bench-pool` | 113,668 MOFs | hMOF (Wilmer 2012 / LLM4Mat-Bench) | table + Robocrystallographer text | 3-class CO₂ adsorption | preprocessed `hmof_pool` | [link](datasets/HMOF.md) | [link](https://iopscience.iop.org/article/10.1088/2632-2153/add3bb/meta) |

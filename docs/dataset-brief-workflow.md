# Using `dataset-brief` on a preprocessed materials dump

This catalog’s first entry — [Nanofiber Composite `io_nn_ready`](../datasets/nanofiber-composite-order-io-nn-ready/README.md) — is a worked example of the **dataset-brief** skill. The skill is read-only on the data. Folder names, paper memory, and README claims are not a substitute for opening records.

## Input

A **complete local path** (file or directory). Dataset names, Hugging Face ids, and URLs are not enough.

Optional: `--index N` or a record id. Default sample is index 0 of the primary table (train split if present).

Nanofiber path used here (host `10.120.17.1`):

```
/sde/shz/data/Nanofiber_ORDER/io_nn_ready
```

## Inspect

```bash
python3 "$SKILL_DIR/scripts/inspect_dataset.py" "<path>"
python3 "$SKILL_DIR/scripts/inspect_dataset.py" "<path>/jsonl/records.jsonl" --index 0
```

Then:

1. Read every path in `inventory.docs` (README, dataset card, `dataset_info.json`). Those are **claims**.
2. Treat `table` / `sample` as **observation**. If they disagree, trust observation.
3. If `sample.image_path` is set, or a field is an existing image path, open that image and describe the pixels — not the filename.
4. If a parser was skipped for a missing library and that file is the primary table, install the library and rerun. Do not unpickle `.pkl` / `.joblib`.

Do not skip the script because a README exists. Do not invent columns, dtypes, shapes, splits, or a “typical” example.

## What happened on Nanofiber

| Step | Result |
|---|---|
| Inventory | 33,006 files: 33,000 `.jpg` (two hard-linked trees), 1 CSV, 1 JSONL, 2 JSON, 2 Python |
| `inventory.docs` | empty — no README in the dump |
| Claims file used anyway | `manifest.json` (not in `docs`, but it is the dump’s own metadata) |
| Primary table | `id_index/table.csv`, 235 rows, 17 columns, all numeric, 0 empty cells |
| Sample index 0 | `ID=2`, six process fields, ten tensile targets |
| Parallel JSONL | same 235 records, nested `features` / `targets`, `images` list truncated by the inspector to 12 of 60 paths |
| Image | `id_index/images/preprocessed/2/2_0.jpg` opened: 512×512 RGB JPEG, R=G=B, SEM of a random nonwoven fiber mat |
| Claim vs observation | ORDER README says “~200 samples”; files have **235** labeled IDs. Manifest still prints old `/ai/data/shz/...` paths; files live under `/sde/shz/...`. |

Replay:

```bash
python3 inspect_dataset.py /sde/shz/data/Nanofiber_ORDER/io_nn_ready --index 0
python3 inspect_dataset.py /sde/shz/data/Nanofiber_ORDER/io_nn_ready/jsonl/records.jsonl --index 0
python3 inspect_dataset.py /sde/shz/data/Nanofiber_ORDER/io_nn_ready --index 195   # ID 199
python3 inspect_dataset.py /sde/shz/data/Nanofiber_ORDER/io_nn_ready --index 211   # ID 215
```

## Infer the task (from fields, docs secondary)

- **Unit of observation:** one electrospun membrane (`ID`), not one tensile-test row.
- **Population:** labeled IDs 2–240 with five image-only IDs dropped; source is ORDER / MatMCL nanofiber (Wu et al., npj Comput. Mater. 2025) as quoted in `ORDER/README.md`.
- **Task type:** multimodal multi-output **regression**.
- **Input vs target vs auxiliary:** process table + SEM bag → ten tensile floats; `ID`, `n_crops`, image paths are auxiliary.

Sibling folders (`table/gen`, `table/sgpt`, `table/retrieval_gallery`, `images/gen`) are other tasks sharing the same materials. They are named in the brief and not treated as the primary task.

## Brief order (required)

The entire user-facing brief is written in this order:

1. **What the data is about** — unit, population, labels, splits, leakage, missingness, imbalance. Ground every claim in an observed field or a quoted doc sentence.
2. **Task input and output** — types, shapes, keys, encoding, ranges, missingness. Nested JSON as a typed schema. Tensors as `dtype` + `shape`.
3. **Case study** — locator, replay command, raw record (keep inspector truncation), walkthrough of every field, description of inspected pixels.

Partial inspection goes under **Limits**. That is the body of [the Nanofiber entry](../datasets/nanofiber-composite-order-io-nn-ready/README.md). Extra benchmark notes (native splits, anisotropy, loader contract) follow the required three sections and are labeled as such.

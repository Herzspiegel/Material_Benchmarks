# Material Benchmarks for Data Scientists

A catalog of **multimodal materials datasets** used to test prediction and uncertainty-quantification methods.

## Material data benchmarks

| ID | Dataset | Dump | Modalities | n | Task | Brief | Original Link |
|---|---|---|---|---|---|---|---|
| `nanofiber-composite-order-io-nn-ready` | Nanofiber Composite (ORDER / MatMCL) | preprocessed `io_nn_ready` | table + SEM image bags | 235 membranes | 10-way tensile regression | [datasets/nanofiber-composite-order-io-nn-ready](datasets/nanofiber-composite-order-io-nn-ready/README.md) | [link](https://www.nature.com/articles/s41524-025-01767-3) |

## How an entry is produced

1. Require a complete on-disk path (not a Hugging Face id).
2. Run `inspect_dataset.py` on that path (and on JSONL if it is a parallel layout).
3. Read `inventory.docs` as **claims**. Treat `table` / `sample` as **observation**. If they disagree, keep observation.
4. If a record points at an image, open the pixels and describe what is depicted.
5. Write the brief in a fixed order: (1) what the data is about, (2) task input/output with types and shapes, (3) case study of one real record.

Worked example: [Nanofiber `io_nn_ready`](datasets/nanofiber-composite-order-io-nn-ready/README.md). Workflow notes: [docs/dataset-brief-workflow.md](docs/dataset-brief-workflow.md).

## License

This catalog’s text is Apache-2.0 (see `LICENSE`). Underlying datasets keep their own licenses (Figshare / paper / code repos cited in each brief). The catalog does **not** republish raw images or tables.

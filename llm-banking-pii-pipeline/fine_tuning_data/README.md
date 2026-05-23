# Supplementary Data

This directory packages the data and benchmark outputs needed to inspect or replicate the reported fine-tuning and evaluation workflow.

## Directory structure

| Path | Contents |
| --- | --- |
| `fine_tuning_dataset/` | The synthetic token-classification corpus used for model fine-tuning, split into training, validation, and test sets. |
| `benchmark_dataset/` | The frozen human-reviewed benchmark set used for external evaluation. |
| `benchmark_comparison/` | Benchmark outputs comparing `cahya/bert-base-indonesian-NER`, `SEA-LION`, and the fine-tuned IndoBERT model on the benchmark dataset. |

## Included files

### `fine_tuning_dataset/`

| File | Description |
| --- | --- |
| `training.jsonl` | Training split (`8,522` records) |
| `validation.jsonl` | Validation split (`1,048` records) |
| `test.jsonl` | Internal synthetic test split (`1,108` records) |
| `dataset_summary.json` | Label list, split sizes, and per-category counts |
| `README.md` | Reader-facing description of the synthetic fine-tuning corpus |

### `benchmark_dataset/`

| File | Description |
| --- | --- |
| `benchmark_cases.jsonl` | Frozen benchmark cases in JSONL format (`320` cases) |
| `benchmark_cases.csv` | CSV rendering of the same benchmark cases |
| `README.md` | Reader-facing description of the benchmark set |

### `benchmark_comparison/`

| File | Description |
| --- | --- |
| `overall_summary.csv` | Overall benchmark metrics per model |
| `category_summary.csv` | Per-category benchmark metrics per model |
| `case_level_results.csv` | Per-case benchmark output in CSV format |
| `case_level_results.json` | Per-case benchmark output in JSON format |
| `style_slice_formal_vs_noisy.csv` | Formal-vs-noisy slice summary derived from the benchmark results |
| `README.md` | Reader-facing description of the benchmark protocol, metrics, formulas, and result files |

## Notes for replication

1. The synthetic fine-tuning corpus and the reviewed benchmark set are separate resources.
2. The benchmark set included here is the frozen external evaluation set, not part of the synthetic train/validation/test split.
3. The file names in this supplementary package are intentionally simplified for readers; they correspond to the final dataset and benchmark artifacts used in the reported experiment.

# Benchmark Comparison

## Overview

This directory contains the benchmark outputs comparing three models on the frozen benchmark set in `../benchmark_dataset/`:

1. `cahya/bert-base-indonesian-NER`
2. `SEA-LION`
3. the fine-tuned IndoBERT model

The benchmark evaluates short Indonesian transactional replies for two target entity types:

- `PER`: person name
- `ADDR`: address or address detail

## Files

| File | Description |
| --- | --- |
| `overall_summary.csv` | Overall metrics per model |
| `category_summary.csv` | Metrics per model and per category |
| `case_level_results.csv` | Case-level results in CSV format |
| `case_level_results.json` | Case-level results in JSON format |
| `style_slice_formal_vs_noisy.csv` | Metrics recomputed for formal and noisy subsets |

## Models compared

| Model label | Underlying model |
| --- | --- |
| Fine-tuned IndoBERT | `indobenchmark/indobert-base-p1` fine-tuned on the synthetic corpus in `../fine_tuning_dataset/` |
| SEA-LION | `aisingapore/Gemma-SEA-LION-v3-9B-IT-GGUF` |
| cahya/bert-base-indonesian-NER | Public Indonesian token-classification baseline |

## Evaluation protocol

For each benchmark case:

1. the same `text` is sent to each model/service
2. person predictions are compared against `expected_persons`
3. address predictions are compared against `expected_addresses`
4. light normalization is applied before comparison:
   - lowercase conversion
   - whitespace cleanup
   - trimming of edge punctuation such as commas or periods
5. counts are aggregated into true positives, false positives, and false negatives

The comparison is label-specific:

- a wrong `PER` prediction contributes to `PER` false positives
- a missed gold `PER` contributes to `PER` false negatives
- the same logic applies to `ADDR`

## Variables and formulas

### Base counts

- `TP`: true positives
- `FP`: false positives
- `FN`: false negatives

These counts are computed separately for person and address labels.

### Precision

Precision measures how many predicted entities are correct.

`Precision = TP / (TP + FP)`

### Recall

Recall measures how many gold entities were successfully recovered.

`Recall = TP / (TP + FN)`

### F1 score

F1 is the harmonic mean of precision and recall.

`F1 = 2 x Precision x Recall / (Precision + Recall)`

Equivalent form using counts:

`F1 = 2 x TP / (2 x TP + FP + FN)`

### Full exact match rate

This metric measures the proportion of benchmark cases for which the complete `PER` output and complete `ADDR` output both match the gold annotations exactly after normalization.

`Full exact match rate = number of cases with exact full match / total number of cases`

### Latency metrics

- `mean_process_time_ms`: arithmetic mean latency across cases
- `median_process_time_ms`: median latency across cases
- `p95_process_time_ms`: latency threshold below which 95% of cases finish

## Overall benchmark results

The following values are taken from `overall_summary.csv`.

| Model | Cases | Person Precision | Person Recall | Person F1 | Address Precision | Address Recall | Address F1 | Full exact match | Mean ms | Median ms | P95 ms |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Fine-tuned IndoBERT | 320 | 0.9296 | 0.9429 | 0.9362 | 0.8730 | 0.9167 | 0.8943 | 0.9406 | 36.782 | 34.106 | 59.249 |
| SEA-LION | 320 | 0.8582 | 0.8643 | 0.8612 | 0.9664 | 0.9583 | 0.9623 | 0.9250 | 6913.415 | 6642.373 | 11048.758 |
| cahya/bert-base-indonesian-NER | 320 | 0.6497 | 0.8214 | 0.7256 | 0.3133 | 0.2167 | 0.2562 | 0.5781 | 59.035 | 56.596 | 78.669 |

## Formal-vs-noisy slice results

`style_slice_formal_vs_noisy.csv` provides a derived slice analysis. It groups selected benchmark categories into:

- `formal`
- `noisy`

The values below are included for convenience.

### Formal subset

| Model | Cases | Person F1 | Address F1 | Full exact match | Mean ms |
| --- | ---: | ---: | ---: | ---: | ---: |
| Fine-tuned IndoBERT | 180 | 0.9016 | 0.8780 | 0.9222 | 38.372 |
| SEA-LION | 180 | 0.7213 | 0.9625 | 0.8944 | 7109.179 |
| cahya/bert-base-indonesian-NER | 180 | 0.6906 | 0.2727 | 0.5389 | 59.139 |

### Noisy subset

| Model | Cases | Person F1 | Address F1 | Full exact match | Mean ms |
| --- | ---: | ---: | ---: | ---: | ---: |
| Fine-tuned IndoBERT | 80 | 0.9250 | 0.8571 | 0.9375 | 33.093 |
| SEA-LION | 80 | 0.9500 | 0.9231 | 0.9500 | 7018.840 |
| cahya/bert-base-indonesian-NER | 80 | 0.6458 | 0.1176 | 0.5750 | 58.110 |

## Category-level output

`category_summary.csv` reports the same family of metrics for each benchmark category and each model. It can be used to inspect:

- which categories are easiest or hardest for each model
- whether errors cluster in person, address, or control cases
- how noisy categories compare with formal categories

Each row contains:

- `service`
- `category`
- `cases`
- `person_precision`
- `person_recall`
- `person_f1`
- `address_precision`
- `address_recall`
- `address_f1`
- `full_exact_match_rate`
- latency statistics

## Case-level output

The case-level files (`case_level_results.csv` and `case_level_results.json`) provide the rawest benchmark view. Each row contains:

- benchmark identifiers
- the original input text
- gold person and address lists
- predicted person and address lists
- TP/FP/FN counts for both labels
- exact-match indicators
- per-case latency

These files are the best starting point for manual error inspection or secondary statistical analysis.

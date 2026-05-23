# Benchmark Dataset

## Overview

This directory contains the frozen human-reviewed benchmark set used for external evaluation.

The benchmark has **320 cases** and is separate from the synthetic fine-tuning corpus. It is intended to evaluate model behavior on realistic short transaction replies after training and checkpoint selection are complete.

## Files

| File | Purpose |
| --- | --- |
| `benchmark_cases.jsonl` | Primary benchmark source file |
| `benchmark_cases.csv` | CSV rendering of the same benchmark set |

Both files contain the same benchmark cases.

## Record format

Each benchmark case includes:

- `case_id`: unique benchmark identifier
- `category`: benchmark category
- `text`: raw benchmark input
- `expected_persons`: gold person spans
- `expected_addresses`: gold address spans

## Corpus composition

| Item | Value |
| --- | ---: |
| Total cases | 320 |
| Categories | 16 |
| Cases with gold `PER` | 140 |
| Cases with gold `ADDR` | 120 |
| No-target control cases | 60 |

The benchmark is single-utterance and single-case based. It is intended to test short, realistic transactional responses rather than long-form documents.

## Benchmark categories

| Category | Cases | Description |
| --- | ---: | --- |
| `short-person-direct` | 20 | Direct person-name replies without additional context |
| `short-person-answer-formal` | 20 | Formal answer-style person replies such as account-owner or recipient-name responses |
| `short-person-answer-noisy` | 20 | Nonstandard or abbreviated person-name replies |
| `short-person-honorific` | 20 | Person-name replies containing honorifics |
| `short-person-rek-owner` | 20 | Person-name replies coupled with account numbers |
| `short-person-transfer-noisy` | 20 | Noisy transfer-style person expressions |
| `short-person-roadlike` | 20 | Person names that resemble street or road-name surfaces |
| `short-address-raw-abbrev` | 20 | Raw abbreviated addresses containing forms such as `Jl.`, `Jln.`, `Gg.`, or `No.` |
| `short-address-raw-full` | 20 | Raw full-form addresses such as `Jalan ...` or `Gang ...` |
| `short-address-block-unit` | 20 | Block/unit/apartment/ruko/kavling address forms |
| `short-address-answer-formal` | 20 | Formal answer-style address replies |
| `short-address-answer-noisy` | 20 | Nonstandard or abbreviated address replies |
| `short-address-roadlike` | 20 | Address expressions containing road-like names that resemble person names |
| `short-control-transfer-fields` | 20 | Non-target transfer fields such as bank names, amounts, or account numbers |
| `short-control-payment-fields` | 20 | Non-target payment fields such as IDPEL or payment instructions |
| `short-control-noisy` | 20 | Noisy non-target controls |

## Relationship to the fine-tuning dataset

This benchmark set is **not** part of the synthetic train/validation/test split.

Its role is different:

- the synthetic corpus in `../fine_tuning_dataset/` is used for training and internal synthetic evaluation
- this benchmark is used for final external evaluation

## Leakage control

The synthetic corpus generator filtered out any training candidate whose raw `text` exactly matched a benchmark case in this reviewed set.

This means:

- the benchmark set was held out from training
- the benchmark set was held out from validation-based checkpoint selection
- exact sentence overlap between the synthetic corpus and this benchmark set was removed

## Recommended evaluation use

To reproduce the reported external evaluation:

1. train and select the model using the synthetic fine-tuning corpus
2. keep this benchmark untouched during training
3. run the final model on `benchmark_cases.jsonl`
4. compare predicted `PER` and `ADDR` spans against `expected_persons` and `expected_addresses`

# Fine-Tuning Dataset

## Overview

This directory contains the synthetic token-classification dataset used to fine-tune the IndoBERT model for two target entity types:

- `PER`: person name
- `ADDR`: address or address detail

The dataset contains **10,678 unique records** and is split into:

| Split | Records | Share |
| --- | ---: | ---: |
| Training | 8,522 | 79.81% |
| Validation | 1,048 | 9.81% |
| Test | 1,108 | 10.38% |
| **Total** | **10,678** | **100.00%** |

## Files

| File | Purpose |
| --- | --- |
| `training.jsonl` | Used to update model weights during fine-tuning |
| `validation.jsonl` | Used for epoch-wise evaluation and best-checkpoint selection |
| `test.jsonl` | Held-out internal synthetic test split |
| `dataset_summary.json` | Split sizes, label list, and per-category counts |

## Record format

Each JSONL line contains a single training example with:

- `id`: unique record identifier
- `category`: category name used by the generator
- `text`: raw training utterance
- `entities`: labeled spans with `start`, `end`, `label`, and `text`

The label space is:

- `O`
- `B-PER`
- `I-PER`
- `B-ADDR`
- `I-ADDR`

This is the standard BIO token-labeling scheme:

- `B-*` marks the first token of an entity
- `I-*` marks subsequent tokens of the same entity
- `O` marks non-entity tokens

## Dataset purpose

The dataset was constructed for short Indonesian transactional language, with emphasis on:

1. transfer instructions and recipient naming
2. electricity-payment requests and address selection
3. short replies and fragmentary follow-up messages
4. ambiguity between person names, bank carriers, and road-like names

## Variation strategy

The synthetic generator introduces variation at three levels so the model does not only see one narrow template family.

### 1. Sentence-level variation

The same target type is expressed through multiple utterance forms, including:

- formal instructions
- answer-style administrative replies
- informal commands
- short replies
- fragmentary or incomplete expressions
- alternative word orders
- contrastive expressions such as `A, not B`

Examples of sentence framing included in the corpus:

- direct transfer instructions
- bank-aware recipient specification
- owner/account phrasing
- payment-location selection
- confirmation-like short replies
- non-target control replies

### 2. Entity-level variation

The entity surface forms are generated with diverse person and address shapes.

#### Person variation

- single-word names
- two-word names
- three-word names
- names with honorifics
- names that overlap with bank names (e.g., `Mega`, `Permata`)
- names that resemble street or historical-person road names
- names appearing in owner/account contexts

#### Address variation

- city-level references
- partial location references
- full road addresses
- block/unit/apartment/ruko/kavling forms
- road-like addresses
- abbreviated address markers such as `Jl.`, `Jln.`, `Gg.`, and `No.`
- full-form markers such as `Jalan`, `Gang`, and `Nomor`

### 3. Noise-level variation

The generator also injects realistic chat-style noise, including:

- lowercase text
- omitted punctuation
- shortened expressions such as `rek`, `rmh`, `id plg`
- colloquial transfer/payment phrases
- informal suffixes such as `-in`
- typo-like or nonstandard forms such as `kiyimin`, `krim`, and `byrin`

## Category families

The dataset contains **96 active categories**. The generator groups them into five high-level families:

| Category family | Cases | Main function |
| --- | ---: | --- |
| `transfer-person-*` | 2,400 | Person extraction in transfer flows |
| `transfer-control-*` | 769 | Non-target transfer controls to suppress false positives |
| `listrik-*`, `noisy-payment-*`, `payment-*` | 3,133 | Person/address extraction and controls in electricity-payment flows |
| `transfer-bank-ambiguity-*`, `transfer-bank-carrier-person-*` | 3,768 | Ambiguity resolution between person names and bank carriers |
| `training-short-*` | 608 | Short-reply coverage for formal, noisy, person, address, and control cases |
| **Total** | **10,678** | Full synthetic corpus |

The full per-category counts are provided in `dataset_summary.json`.

## Split procedure

The dataset was generated first and then split by category into:

- 80% training
- 10% validation
- 10% test

A fixed random seed was used for reproducible shuffling.

## Deduplication and leakage control

The generator applied exact-text deduplication before the final split:

1. if two generated records had identical `text`, only the first one was kept
2. the train/validation/test split was performed only after deduplication
3. exact-text overlap with the frozen reviewed benchmark set was removed before final export

As a result:

- total unique texts = `10,678`
- exact overlap with the reviewed benchmark set = `0`
- recorded span-offset errors = `0`

## How to use this dataset

For replication:

1. train the model using `training.jsonl`
2. evaluate checkpoints on `validation.jsonl`
3. report internal synthetic test results on `test.jsonl`
4. keep the external benchmark in `../benchmark_dataset/` separate from model selection

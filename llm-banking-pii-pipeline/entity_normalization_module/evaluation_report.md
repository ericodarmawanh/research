# Entity Normalization Module: Supplementary Evaluation Report

## What This File Is For

This report is intended for readers who want to understand:

- what the Entity Normalization Module does
- what was evaluated
- where the evaluation dataset came from
- how to read the supplementary JSON and CSV files
- how the reported match-rate improvement was computed
- which cases improved and which failure types remained

## The Four Supplementary Files In This Folder

This supplementary package contains exactly four files:

- `evaluation_dataset.json`: the benchmark input dataset used by the Entity Normalization Module evaluator
- `evaluation_results.json`: the structured evaluation output
- `evaluation_results.csv`: the tabular export of the structured evaluation output
- `evaluation_report.md`: this explanatory report

## What The Entity Normalization Module Does

The Entity Normalization Module is a lightweight post-processing layer applied after the NER stage and before the text is passed to the LLM.

Its purpose is not to detect new entities and not to recover the original semantic label of `person` or `location`.

Its narrower role is to repair structurally fragmented NER outputs so that one sensitive reference does not reach the LLM as two or more misleading placeholder fragments.

The module is therefore evaluated as a structural repair layer.

## What Was Evaluated

This evaluation tests whether the Entity Normalization Module improves full exact match on the canonicalized redaction output.

More specifically, each benchmark case compares:

- the baseline canonical output obtained directly from the pre-normalization NER redaction
- the post-processed canonical output obtained after the Entity Normalization Module is applied
- the expected canonical output derived from the benchmark gold entities

The main question is:

Can post-processing improve exact agreement with the expected canonicalized output on the 320 benchmark cases?

## Important Provenance Note

The benchmark input in this supplementary package originates from a previously completed NER benchmark.

That earlier benchmark produced a file named `benchmark_results.json`, which contained 320 benchmark cases together with the upstream NER redaction output and supporting benchmark metadata.

For the Entity Normalization Module evaluation, that earlier benchmark artifact was reused as the benchmark input.

In other words:

- the source benchmark had already been completed earlier
- its output artifact was reused as input to the Entity Normalization Module evaluator
- the evaluator then measured whether normalization improved exact match against the expected canonical output

This is why the supplementary `evaluation_dataset.json` should be understood as a cleaned evaluation input derived from the earlier benchmark artifact, not as a brand-new independently collected dataset.

## Why The Supplementary Dataset Was Renamed

In the original evaluation folder, the input file was named `benchmark_results.json`.

That name reflects the history of the file, but it is confusing when the file is packaged for supplementary reading, because it sounds like an output file rather than an input file.

For the supplementary package, the same benchmark input has therefore been repackaged as `evaluation_dataset.json`.

The supplementary dataset keeps only the fields that are actually consumed by the Entity Normalization Module evaluator:

- `case_id`
- `category`
- `text`
- `redacted_text`
- `expected_persons`
- `expected_addresses`

This makes the role of the file clearer for paper readers.

## How The Evaluator Works

The evaluation procedure is straightforward.

For each benchmark case:

1. the evaluator reads the original text and the expected entities from the benchmark input
2. the evaluator converts the gold entities into an `expected_canonical` string using the neutral placeholder `[token-ent]`
3. the evaluator converts the raw NER redaction into a `baseline_canonical` string
4. the evaluator applies the Entity Normalization Module to obtain a `post_processed_canonical` string
5. the evaluator checks whether the baseline string exactly matches the expected canonical string
6. the evaluator checks whether the post-processed string exactly matches the expected canonical string

The evaluation therefore compares exact string equality after canonicalization.

## What “Full Exact Match” Means Here

In this evaluation, the reported improvement from `94.06%` to `94.69%` refers to full exact match on the canonicalized output string.

That is why the result JSON uses the field names:

- `baseline_match_rate`
- `post_processed_match_rate`

These rates measure exact match between the produced canonical string and the expected canonical string for the whole case.

So, in the context of this module, it is accurate to describe the result as an improvement in full exact match over canonicalized redaction output.

## What The Dataset File Contains

Each object in `evaluation_dataset.json` is one benchmark case.

Fields:

- `case_id`: unique case identifier
- `category`: benchmark category
- `text`: the original user text
- `redacted_text`: the upstream NER redaction used as baseline input to the module
- `expected_persons`: gold list of person strings for the case
- `expected_addresses`: gold list of address strings for the case

The evaluator uses `expected_persons` and `expected_addresses` to build the expected canonical target for the case.

## What The Results JSON Contains

The `evaluation_results.json` file contains:

- a top-level `summary` object
- a `cases` array with one evaluated row per benchmark case

### Summary fields

- `generated_at`
- `total_cases`
- `baseline_match_count`
- `post_processed_match_count`
- `baseline_match_rate`
- `post_processed_match_rate`
- `improved_case_count`
- `remaining_failure_count`

### Case-level fields

- `case_id`
- `category`
- `text`
- `redacted_text`
- `expected_canonical`
- `baseline_canonical`
- `post_processed_canonical`
- `baseline_matched`
- `post_processed_matched`

## What The CSV File Contains

The CSV file is a flat export of the same case-level results.

Columns:

- `case_id`
- `category`
- `text`
- `redacted_text`
- `expected_canonical`
- `baseline_canonical`
- `post_processed_canonical`
- `baseline_matched`
- `post_processed_matched`

The CSV is useful for filtering improved cases, remaining failures, and category-specific patterns.

## Evaluation Size And Main Result

The benchmark contains:

- `320` total cases
- `301` baseline exact matches
- `303` post-processed exact matches
- `2` improved cases
- `17` remaining post-processed failures

The main rates are:

- baseline exact match: `301 / 320 = 94.06%`
- post-processed exact match: `303 / 320 = 94.69%`

This means the Entity Normalization Module produced a net gain of 2 exactly matched cases on this benchmark.

## Metric Definitions

### 1. Baseline Exact Match Rate

Question:

How often does the unnormalized baseline canonical output exactly match the expected canonical output?

Formula:

`Baseline Exact Match Rate = baseline_match_count / total_cases * 100%`

For this run:

- `301 / 320 = 94.06%`

### 2. Post-Processed Exact Match Rate

Question:

How often does the post-processed canonical output exactly match the expected canonical output?

Formula:

`Post-Processed Exact Match Rate = post_processed_match_count / total_cases * 100%`

For this run:

- `303 / 320 = 94.69%`

### 3. Improved Case Count

Question:

How many cases were incorrect in the baseline output but correct after Entity Normalization Module post-processing?

Formula:

`Improved Case Count = number of cases where baseline_matched = false and post_processed_matched = true`

For this run:

- `2`

### 4. Remaining Failure Count

Question:

How many cases still do not exactly match the expected canonical output after post-processing?

Formula:

`Remaining Failure Count = number of cases where post_processed_matched = false`

For this run:

- `17`

## Category Composition

The 320-case benchmark is evenly distributed across 16 categories, with 20 cases per category.

| Category | Cases |
| --- | ---: |
| `short-address-answer-formal` | 20 |
| `short-address-answer-noisy` | 20 |
| `short-address-block-unit` | 20 |
| `short-address-raw-abbrev` | 20 |
| `short-address-raw-full` | 20 |
| `short-address-roadlike` | 20 |
| `short-control-noisy` | 20 |
| `short-control-payment-fields` | 20 |
| `short-control-transfer-fields` | 20 |
| `short-person-answer-formal` | 20 |
| `short-person-answer-noisy` | 20 |
| `short-person-direct` | 20 |
| `short-person-honorific` | 20 |
| `short-person-rek-owner` | 20 |
| `short-person-roadlike` | 20 |
| `short-person-transfer-noisy` | 20 |
| **Total** | **320** |

The category names already indicate the benchmark emphasis:

- `short-person-*` categories focus on person-name redaction patterns
- `short-address-*` categories focus on address redaction patterns
- `short-control-*` categories are control cases that help verify non-target patterns and non-improving conditions

## Category-Level Exact Match Counts

The table below compares baseline and post-processed exact-match counts within each category.

| Category | Cases | Baseline Match | Post-Processed Match |
| --- | ---: | ---: | ---: |
| `short-address-answer-formal` | 20 | 19 | 20 |
| `short-address-answer-noisy` | 20 | 18 | 19 |
| `short-address-block-unit` | 20 | 13 | 13 |
| `short-address-raw-abbrev` | 20 | 20 | 20 |
| `short-address-raw-full` | 20 | 20 | 20 |
| `short-address-roadlike` | 20 | 20 | 20 |
| `short-control-noisy` | 20 | 20 | 20 |
| `short-control-payment-fields` | 20 | 19 | 19 |
| `short-control-transfer-fields` | 20 | 20 | 20 |
| `short-person-answer-formal` | 20 | 20 | 20 |
| `short-person-answer-noisy` | 20 | 17 | 17 |
| `short-person-direct` | 20 | 20 | 20 |
| `short-person-honorific` | 20 | 16 | 16 |
| `short-person-rek-owner` | 20 | 19 | 19 |
| `short-person-roadlike` | 20 | 20 | 20 |
| `short-person-transfer-noisy` | 20 | 20 | 20 |

The two net improvements occur in:

- `short-address-answer-formal`
- `short-address-answer-noisy`

## Improved Cases

This run contains exactly two improved cases.

### 1. `V15R206`

- category: `short-address-answer-formal`
- baseline redaction: `Alamat rumahnya di [LOC1], [LOC2].`
- post-processed canonical: `Alamat rumahnya di [token-ent].`
- expected canonical: `Alamat rumahnya di [token-ent].`

This case shows the intended behavior of the module: adjacent or closely related address fragments are collapsed into one neutral canonical reference.

### 2. `V15R226`

- category: `short-address-answer-noisy`
- baseline redaction: `rmh yg mau dibayar di [LOC1], [LOC2]`
- post-processed canonical: `rmh yg mau dibayar di [token-ent]`
- expected canonical: `rmh yg mau dibayar di [token-ent]`

This is the same structural gain in a noisier language style.

## What The Remaining Failures Mean

After post-processing, `17` cases still do not exactly match the expected canonical output.

These remaining failures are useful because they show the intended boundary of the module.

The Entity Normalization Module is conservative.

It improves multipart fragmentation patterns, but it does not aggressively attempt to solve:

- pure NER misses
- false positives
- single-placeholder underspan cases
- full semantic boundary reconstruction
- aggressive trimming of honorifics, prefixes, or contextual tokens

This conservative design is consistent with the role of the module as a structural repair layer rather than a second-stage semantic span reconstructor.

## How To Recheck The Numbers From The Supplementary Files

Readers can verify the reported numbers directly from the four supplementary files.

### Check Dataset Size

Open `evaluation_dataset.json` and count the number of objects.

You should get:

- `320`

### Check Results Size

Open `evaluation_results.json` and count the number of objects in `cases`.

You should also get:

- `320`

### Check The Top-Level Summary

Open `evaluation_results.json` and read the `summary` object.

You should see:

- `total_cases = 320`
- `baseline_match_count = 301`
- `post_processed_match_count = 303`
- `baseline_match_rate = 0.940625`
- `post_processed_match_rate = 0.946875`
- `improved_case_count = 2`
- `remaining_failure_count = 17`

### Recompute Baseline Exact Match Rate

In `evaluation_results.csv`:

- count rows where `baseline_matched = true`
- divide by total rows

You should get:

- `301 / 320 = 94.06%`

### Recompute Post-Processed Exact Match Rate

In `evaluation_results.csv`:

- count rows where `post_processed_matched = true`
- divide by total rows

You should get:

- `303 / 320 = 94.69%`

### Recompute Improved Case Count

In `evaluation_results.csv` or `evaluation_results.json`:

- count cases where `baseline_matched = false`
- among those, count cases where `post_processed_matched = true`

You should get:

- `2`

The two cases are:

- `V15R206`
- `V15R226`

### Recompute Remaining Failures

In `evaluation_results.csv` or `evaluation_results.json`:

- count cases where `post_processed_matched = false`

You should get:

- `17`

## Suggested Reading Order

For a paper reader, the simplest reading order is:

1. `evaluation_report.md`
2. `evaluation_dataset.json`
3. `evaluation_results.json`
4. `evaluation_results.csv`

The report explains the provenance and metric definitions.

The dataset file shows the benchmark input actually consumed by the evaluator.

The results JSON shows the canonical comparison output and the aggregate counts.

The CSV makes it easier to inspect improved cases and remaining failures.
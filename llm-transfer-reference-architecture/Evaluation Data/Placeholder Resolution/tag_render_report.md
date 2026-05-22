# Tag Render Evaluation Results For Readers

## What This File Is For

This file explains the tag-render evaluation (placeholder resolution), which is written for readers who want to:

- understand what was tested
- understand what each metric means
- check the reported numbers by using the supplementary files

This file does not replace the raw supplementary data.

The main supplementary files are:

- `tag_render_dataset.json`: the input evaluation dataset. It contains the original test cases, including the text before rendering, the draft data, the expected final output, and the expected case class.
- `tag_render_results.json`: the full structured evaluation output. It contains the actual rendered output for each case, the per-case flags used for scoring, the final class, and the aggregate summary.
- `tag_render_results.csv`: the tabular export of the evaluation results. It is useful for readers who want to inspect or sort the cases quickly in spreadsheet software.

## What Was Tested

This evaluation tests only one small part of the system.

It does **not** test the whole chatbot.

It only tests the step that changes output tags into the final text shown to the user.

Example:

- text before rendering: `Transfer to [ACC_NAME].`
- text after rendering: `Transfer to Doni.`

## What A Tag Means

A **tag** is a placeholder inside the text.

The supported tags in this evaluation are:

- `[ACC_NAME]` = destination account name
- `[BANK_NAME]` = destination bank name
- `[ACC_NUM]` = destination account number
- `[TRF_AMT]` = transfer amount

A supported tag is called a **known tag**.

A tag that is not supported by the renderer is called an **unknown tag**.

Example:

- known tag: `[ACC_NAME]`
- unknown tag: `[UNKNOWN_TAG]`

## What Draft State Means

The **draft state** is the transfer data available when rendering starts.

Examples:

- account name = `Doni`
- bank name = `BCA`
- transfer amount = `25000`

If the text contains a known tag but the needed value is missing from the draft state, the renderer uses a safe replacement phrase.

This is called **fallback**.

Examples of fallback phrases used in this evaluation are:

- `rekening tujuan`
- `bank tujuan`
- `nominal transfer`
- `nomor rekening tujuan`

## What The Dataset File Contains

Each item in `tag_render_dataset.json` is one evaluation case.

Main fields:

- `id`: case ID
- `category`: case type
- `llm_output`: text before rendering
- `draft`: available transfer data
- `expected_output`: correct final text
- `expected_class`: expected case result type

## What The Results File Contains

Each item in `tag_render_results.json` is one evaluated case.

Main fields:

- `case_id`: case ID
- `category`: case type
- `llm_output`: text before rendering
- `draft_state`: the full draft data used by the renderer
- `expected_output`: correct final text
- `rendered_output`: actual final text produced by the renderer
- `known_tags_present`: list of supported tags found in the text
- `unknown_tags_present`: list of unsupported tags found in the text
- `fallback_tags_used`: tags that used fallback phrases
- `has_known_tags`: whether the case contains at least one supported tag
- `all_known_tags_resolved`: whether all supported tags were rendered correctly
- `fallback_used`: whether fallback was used
- `fallback_correct`: whether the fallback text was correct
- `contains_unknown_tags`: whether the case contains unknown tags
- `unknown_tag_handled_safely`: whether unknown tags stayed stable and did not break rendering
- `exact_output_match`: whether the final output exactly matched the expected output
- `final_class`: the final class assigned by the evaluator
- `expected_class`: the expected class for the case
- `expectation_match`: whether `final_class` equals `expected_class`

## Case Classes

The evaluator uses five case classes.

### Fully Resolved

All known tags were replaced correctly.

No fallback phrase was needed.

### Resolved With Fallback

Rendering succeeded, but one or more values were missing, so fallback text was used.

### Stable With Unknown Tag

The case contains unknown tags.

Those unknown tags stayed unchanged, and they did not break the rendering of valid tags.

### Render Failure

The final rendered text did not match the expected text.

### No Tag Content

The text contains no tags and passes through unchanged.

## Dataset Composition

The dataset has 36 cases.

| Category | Meaning | Cases |
| --- | --- | ---: |
| `full_resolution` | all needed values are present | 9 |
| `partial_draft_fallback` | some values are present and some are missing | 9 |
| `all_fallback` | all supported tags must use fallback text | 6 |
| `unknown_tag` | the text contains unknown tags | 5 |
| `mixed_known_and_unknown_tags` | the text contains both known and unknown tags | 4 |
| `no_tag_content` | the text contains no tags | 3 |
| **Total** |  | **36** |

## Metric Definitions

### 1. Tag Resolution Success Rate (TRSR)

This metric checks cases that contain one or more known tags.

Question:

Did the renderer replace all known tags correctly?

Formula:

`TRSR = correct_known_tag_cases / valid_tag_cases * 100%`

### 2. Fallback Correctness Rate (FCR)

This metric checks only cases where fallback was used.

Question:

When fallback was needed, was the fallback phrase correct?

Formula:

`FCR = fallback_correct_cases / fallback_cases * 100%`

### 3. Unknown Tag Stability Rate (UTSR)

This metric checks only cases that contain unknown tags.

Question:

Did the renderer stay stable, keep unknown tags unchanged, and still handle known tags correctly?

Formula:

`UTSR = stable_unknown_tag_cases / unknown_tag_cases * 100%`

### 4. Output Fidelity Rate (OFR)

This metric checks all cases.

Question:

Did the final rendered output exactly match the expected output?

Formula:

`OFR = exact_output_match_cases / total_cases * 100%`

### 5. Expectation Match Rate

This metric checks whether the final case label matched the expected case label.

Question:

Did the evaluator assign the correct class to the case?

Formula:

`Expectation Match Rate = expectation_match_cases / total_cases * 100%`

## Raw Counts Used For The Final Results

The reported results are based on these raw counts.

| Item | Raw count | Value |
| --- | ---: | ---: |
| Total cases | 36 | 36 |
| Valid-tag cases | 32 | 32 |
| Fallback cases | 15 | 15 |
| Unknown-tag cases | 9 | 9 |
| Fully Resolved | 9 / 36 | 25.00% of all cases |
| Resolved With Fallback | 15 / 36 | 41.67% of all cases |
| Stable With Unknown Tag | 9 / 36 | 25.00% of all cases |
| Render Failure | 0 / 36 | 0.00% of all cases |
| No Tag Content | 3 / 36 | 8.33% of all cases |
| TRSR | 32 / 32 | 100.00% |
| FCR | 15 / 15 | 100.00% |
| UTSR | 9 / 9 | 100.00% |
| OFR | 36 / 36 | 100.00% |
| Expectation Match Rate | 36 / 36 | 100.00% |

## Why The Denominators Are Different

Not every metric uses all 36 cases.

- `TRSR` uses only the 32 cases with known tags
- `FCR` uses only the 15 cases where fallback was used
- `UTSR` uses only the 9 cases with unknown tags
- `OFR` uses all 36 cases
- `Expectation Match Rate` uses all 36 cases

This is why the same evaluation can report different denominators for different metrics.

## Category-Level Results

| Category | Cases | Fully Resolved | Resolved With Fallback | Stable With Unknown Tag | Render Failure | No Tag Content | TRSR | FCR | UTSR | OFR |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `all_fallback` | 6 | 0 | 6 | 0 | 0 | 0 | 100.00% | 100.00% | 0.00% | 100.00% |
| `full_resolution` | 9 | 9 | 0 | 0 | 0 | 0 | 100.00% | 0.00% | 0.00% | 100.00% |
| `mixed_known_and_unknown_tags` | 4 | 0 | 0 | 4 | 0 | 0 | 100.00% | 0.00% | 100.00% | 100.00% |
| `no_tag_content` | 3 | 0 | 0 | 0 | 0 | 3 | 0.00% | 0.00% | 0.00% | 100.00% |
| `partial_draft_fallback` | 9 | 0 | 9 | 0 | 0 | 0 | 100.00% | 100.00% | 0.00% | 100.00% |
| `unknown_tag` | 5 | 0 | 0 | 5 | 0 | 0 | 100.00% | 0.00% | 100.00% | 100.00% |

## Simple Examples

### Example A: Full Resolution

- case: `RTR-001`
- input text: `Saya akan membantu transfer ke [ACC_NAME].`
- draft value: `destination_account_name = Doni`
- final output: `Saya akan membantu transfer ke Doni.`

This is a `Fully Resolved` case because the known tag was replaced correctly and no fallback was needed.

### Example B: Fallback

- case: `RTR-009`
- input text: `Saya akan membantu transfer ke [ACC_NAME]. Mohon isi [TRF_AMT].`
- `[ACC_NAME]` has a value in the draft
- `[TRF_AMT]` has no value in the draft
- final output: `Saya akan membantu transfer ke Doni. Mohon isi nominal transfer.`

This is a `Resolved With Fallback` case because fallback text was used for the missing amount.

### Example C: Unknown Tag

- case: `RTR-024`
- input text: `Nominal [TRF_AMT] untuk [UNKNOWN_TAG].`
- known tag: `[TRF_AMT]`
- unknown tag: `[UNKNOWN_TAG]`
- final output: `Nominal Rp 50.000 untuk [UNKNOWN_TAG].`

This is a `Stable With Unknown Tag` case because the known tag was rendered correctly and the unknown tag stayed unchanged.

## How To Verify The Numbers From The Supplementary Files

### Step 1: Check total cases

Count the number of cases in `tag_render_dataset.json` or `tag_render_results.json`.

Expected result: `36`

### Step 2: Check valid-tag cases

In `tag_render_results.json`, select cases where `has_known_tags = true`.

Expected count: `32`

### Step 3: Check fallback cases

In `tag_render_results.json`, select cases where `fallback_used = true`.

Expected count: `15`

### Step 4: Check unknown-tag cases

In `tag_render_results.json`, select cases where `contains_unknown_tags = true`.

Expected count: `9`

### Step 5: Check TRSR

In `tag_render_results.json`:

1. select cases where `has_known_tags = true`
2. count them
3. among those cases, count how many have `all_known_tags_resolved = true`

Expected result:

- denominator = `32`
- numerator = `32`
- `TRSR = 32 / 32 = 100.00%`

### Step 6: Check FCR

In `tag_render_results.json`:

1. select cases where `fallback_used = true`
2. count them
3. among those cases, count how many have `fallback_correct = true`

Expected result:

- denominator = `15`
- numerator = `15`
- `FCR = 15 / 15 = 100.00%`

### Step 7: Check UTSR

In `tag_render_results.json`:

1. select cases where `contains_unknown_tags = true`
2. count them
3. among those cases, count how many have `unknown_tag_handled_safely = true`

Expected result:

- denominator = `9`
- numerator = `9`
- `UTSR = 9 / 9 = 100.00%`

### Step 8: Check OFR

In `tag_render_results.json`:

1. count all cases
2. count how many have `exact_output_match = true`

Expected result:

- denominator = `36`
- numerator = `36`
- `OFR = 36 / 36 = 100.00%`

### Step 9: Check expectation match rate

In `tag_render_results.json`:

1. count all cases
2. count how many have `expectation_match = true`

Expected result:

- denominator = `36`
- numerator = `36`
- `Expectation Match Rate = 36 / 36 = 100.00%`

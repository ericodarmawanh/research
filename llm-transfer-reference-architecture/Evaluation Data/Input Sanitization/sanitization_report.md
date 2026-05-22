# Sanitization Evaluation Results For Readers

## What This File Is For

This file explains the sanitization evaluation for readers who want to:

- understand what was tested
- understand what each metric means
- check the reported numbers by using the supplementary files

The main supplementary files are:

- `sanitization_dataset.json`: the input evaluation dataset. It contains the test cases, the original input text, the case category, and the expected final class for each case.
- `sanitization_results.json`: the full structured evaluation output. It contains the aggregate summary and the per-case results, including detected numbers, sanitized text, parse status, security flags, final class, and latency.
- `sanitization_results.csv`: the tabular export of the evaluation results. It is useful for readers who want to inspect, sort, or filter cases quickly in spreadsheet software.

## What Was Tested

This evaluation tests only the numeric sanitization layer.

It does **not** test the full chatbot.

It only tests the step that detects numeric content in user text, replaces it with placeholders, and records the internal parsing result.

Example:

- text before sanitization: `Transfer 25000 ke Doni`
- text after sanitization: `Transfer [token: x68hczcn] ke Doni`

## What Sanitization Means Here

In this evaluation, **sanitization** means that raw numbers are removed from the text that would be sent forward to the model-facing layer.

The number is replaced with a token such as `[token: x68hczcn]`.

The original number is kept only in the internal token mapping (internal vault).

This means the model-facing text no longer contains the raw numeric string.

## What Normalization Means Here

In this evaluation, **normalization** means that the detected number could also be interpreted safely as a valid numeric value under the current rules.

Examples that are accepted in this evaluation include:

- plain integers such as `25000`
- supported decimals such as `10.5`
- supported decimals such as `25,5`

Examples that are intentionally rejected in this evaluation include:

- thousands separators such as `15.000`
- mixed separators such as `1.234,56`
- split formats such as `1234-5678-9012`

A rejected normalization result is **not** automatically a failure.

If the raw number was still replaced safely and no raw numeric text leaked, the case is treated as a safe rejection.

## What The Dataset File Contains

Each item in `sanitization_dataset.json` is one evaluation case.

Main fields:

- `id`: case ID
- `category`: case type
- `input_text`: original user text before sanitization
- `expected_class`: expected final case class

## What The Results File Contains

Each item in `sanitization_results.json` is one evaluated case.

Main fields:

- `case_id`: case ID
- `category`: case type
- `input_text`: original user text
- `detected_numbers`: numeric substrings detected in the input
- `token_mappings`: token assignment plus parse status, parse reason, and numeric value when available
- `sanitized_text`: text after numeric replacement
- `has_numeric_input`: whether the case contains numeric content
- `sanitization_success`: whether numeric content was contained safely
- `normalization_success`: whether numeric parsing succeeded under current rules
- `clarification_required`: whether the system should ask the user to restate the number in a supported format
- `security_leak_detected`: whether raw numeric text remained in the sanitized output
- `final_class`: final result assigned by the evaluator
- `expected_class`: expected class for the case
- `expectation_match`: whether `final_class` equals `expected_class`
- `tokenization_elapsed_milliseconds`: time spent in the sanitization step

## Case Classes

The evaluator uses four main case classes.

### Fully Successful

The numeric content was sanitized safely and normalization also succeeded.

### Safely Rejected

The numeric content was sanitized safely, but normalization was intentionally rejected.

This means the system should ask for clarification instead of guessing.

### Security Failure

Raw numeric content leaked into the sanitized output, or sanitization containment failed.

### No Numeric Content

The input did not contain numeric content.

These cases are useful for behavior coverage, but they are not part of the core numeric security denominator.

## Dataset Composition

The dataset has 60 cases.

| Category | Meaning | Cases |
| --- | --- | ---: |
| `plain_integer` | simple whole numbers | 8 |
| `decimal_input` | supported decimal numbers | 8 |
| `multi_number_input` | more than one numeric value in one input | 5 |
| `mixed_banking_text` | banking text mixed with numeric values | 8 |
| `long_account_number` | long numeric account-like strings | 4 |
| `unsupported_numeric_format` | unsupported separator patterns | 7 |
| `multiple_separators` | numbers with more than one separator style | 6 |
| `hyphenated_number` | numbers split by hyphens | 3 |
| `slash_separated_number` | numbers split by slashes | 3 |
| `space_separated_number` | numbers split by spaces | 3 |
| `no_number_input` | no numeric content | 5 |
| **Total** |  | **60** |

Two high-level groups matter most for the metrics:

- numeric cases: 55
- no-numeric cases: 5

Within the 55 numeric cases, 22 are unsupported-format cases that should be rejected safely rather than guessed.

## Metric Definitions

### 1. Sanitization Containment Rate (SCR)

This metric checks numeric cases.

Question:

Did the system replace the raw numbers with tokens and keep the sanitized output free from raw numeric leakage?

Formula:

`SCR = contained_numeric_cases / total_numeric_cases * 100%`

### 2. Numeric Normalization Success Rate (NNSR)

This metric also checks numeric cases.

Question:

Did the system both sanitize the number and interpret it successfully under the current parsing rules?

Formula:

`NNSR = normalized_numeric_cases / total_numeric_cases * 100%`

### 3. Safe Clarification Handling Rate (SCHR)

This metric checks only unsupported-format cases.

Question:

When the format was unsupported, did the system still sanitize the text safely and correctly mark the case for clarification?

Formula:

`SCHR = safely_handled_unsupported_cases / total_unsupported_cases * 100%`

### 4. Leakage Incidence Rate (LIR)

This metric checks numeric cases.

Question:

How often did raw numeric content leak into the sanitized output?

Formula:

`LIR = leak_cases / total_numeric_cases * 100%`

### 5. Expectation Match Rate

This metric checks all cases.

Question:

Did the evaluator assign the expected final class to the case?

Formula:

`Expectation Match Rate = expectation_match_cases / total_cases * 100%`

## Raw Counts Used For The Final Results

The reported results are based on these raw counts.

| Item | Raw count | Value |
| --- | ---: | ---: |
| Total cases | 60 | 100.00% |
| Numeric cases | 55 | 91.67% of all cases |
| Unsupported cases | 22 | 40.00% of numeric cases |
| Fully Successful | 33 | 60.00% of numeric cases |
| Safely Rejected | 22 | 40.00% of numeric cases |
| Security Failure | 0 | 0.00% of numeric cases |
| No Numeric Content | 5 | 8.33% of all cases |
| SCR | 55 / 55 | 100.00% |
| NNSR | 33 / 55 | 60.00% |
| SCHR | 22 / 22 | 100.00% |
| LIR | 0 / 55 | 0.00% |
| Expectation Match Rate | 60 / 60 | 100.00% |

Average tokenization latency in this run was `0.106833 ms` per case.

## Why The Denominators Are Different

The denominators are different because each metric answers a different question.

- `SCR` uses all numeric cases, so the denominator is `55`.
- `NNSR` also uses all numeric cases, so the denominator is `55`.
- `SCHR` uses only unsupported-format cases, so the denominator is `22`.
- `LIR` uses all numeric cases, so the denominator is `55`.
- `Expectation Match Rate` uses all cases, so the denominator is `60`.

This is why the report shows:

- `SCR = 55 / 55`
- `NNSR = 33 / 55`
- `SCHR = 22 / 22`
- `LIR = 0 / 55`
- `Expectation Match Rate = 60 / 60`

## Category-Level Results

The category breakdown helps explain where successful normalization happened and where safe rejection happened.

| Category | Cases | Numeric cases | Fully Successful | Safely Rejected | Security Failure | SCR | NNSR | SCHR | LIR |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `decimal_input` | 8 | 8 | 8 | 0 | 0 | 100.00% | 100.00% | 0.00% | 0.00% |
| `hyphenated_number` | 3 | 3 | 0 | 3 | 0 | 100.00% | 0.00% | 100.00% | 0.00% |
| `long_account_number` | 4 | 4 | 4 | 0 | 0 | 100.00% | 100.00% | 0.00% | 0.00% |
| `mixed_banking_text` | 8 | 8 | 8 | 0 | 0 | 100.00% | 100.00% | 0.00% | 0.00% |
| `multi_number_input` | 5 | 5 | 5 | 0 | 0 | 100.00% | 100.00% | 0.00% | 0.00% |
| `multiple_separators` | 6 | 6 | 0 | 6 | 0 | 100.00% | 0.00% | 100.00% | 0.00% |
| `no_number_input` | 5 | 0 | 0 | 0 | 0 | 0.00% | 0.00% | 0.00% | 0.00% |
| `plain_integer` | 8 | 8 | 8 | 0 | 0 | 100.00% | 100.00% | 0.00% | 0.00% |
| `slash_separated_number` | 3 | 3 | 0 | 3 | 0 | 100.00% | 0.00% | 100.00% | 0.00% |
| `space_separated_number` | 3 | 3 | 0 | 3 | 0 | 100.00% | 0.00% | 100.00% | 0.00% |
| `unsupported_numeric_format` | 7 | 7 | 0 | 7 | 0 | 100.00% | 0.00% | 100.00% | 0.00% |

A simple way to read this table is:

- supported formats reached both containment and normalization success
- unsupported formats still reached containment success, but were rejected for normalization on purpose
- no category produced a security failure in this run

## Simple Examples

### Fully Successful Example

- input: `Transfer 25000 ke Doni`
- output: `Transfer [token: x68hczcn] ke Doni`
- result: the raw number was hidden and the value was parsed successfully

### Safely Rejected Example

- input: `Transfer 15.000 ke Doni`
- output: `Transfer [token: e1hpj93h] ke Doni`
- result: the raw number was still hidden safely, but the format was rejected and should trigger clarification

### Mixed Example With One Valid Number And One Unsupported Number

- input: `Tanggal 30/03/2026 transfer ke Doni sebesar 10000`
- output: `Tanggal [token: 03iby5a9] transfer ke Doni sebesar [token: 7of9zojg]`
- result: both numeric strings were hidden safely, but the case still ends as `Safely Rejected` because one detected number (`30/03/2026`) uses an unsupported format and requires clarification

This kind of case is important because the final class is decided at the case level, not separately for each number.

In other words, even if one number is valid, the whole case is still treated as `Safely Rejected` when another detected number remains unsupported.

### No Numeric Content Example

- input: `Saya ingin transfer ke Doni`
- output: `Saya ingin transfer ke Doni`
- result: there was no numeric content to sanitize

## How To Verify The Numbers From The Supplementary Files

A reader can re-check the final numbers directly from the supplementary files.

### Check The Dataset Size

Open `sanitization_dataset.json`.

Count the total number of objects.

You should get `60` cases.

### Check The Top-Level Summary

Open `sanitization_results.json` and read the `summary` object.

You should see:

- `total_cases = 60`
- `numeric_case_count = 55`
- `unsupported_case_count = 22`
- `fully_successful_count = 33`
- `safely_rejected_count = 22`
- `security_failure_count = 0`
- `no_numeric_content_count = 5`
- `scr = 100.0`
- `nnsr = 60.0`
- `schr = 100.0`
- `lir = 0.0`
- `expectation_match_rate = 100.0`

### Recompute SCR From Per-Case Results

In `sanitization_results.json` or `sanitization_results.csv`:

- filter rows where `has_numeric_input = true`
- count how many of those also have `sanitization_success = true` and `security_leak_detected = false`

You should get `55 / 55`.

### Recompute NNSR From Per-Case Results

In `sanitization_results.json` or `sanitization_results.csv`:

- filter rows where `has_numeric_input = true`
- count how many of those have `normalization_success = true`

You should get `33 / 55`.

### Recompute SCHR From Per-Case Results

In `sanitization_results.json` or `sanitization_results.csv`:

- identify the unsupported-format cases
- these correspond to cases that end as safe rejection and require clarification
- count how many of those were still sanitized safely and had no leakage

You should get `22 / 22`.

### Recompute LIR From Per-Case Results

In `sanitization_results.json` or `sanitization_results.csv`:

- filter rows where `has_numeric_input = true`
- count how many of those have `security_leak_detected = true`

You should get `0 / 55`.

### Recompute Expectation Match Rate

In `sanitization_results.json` or `sanitization_results.csv`:

- count all rows where `expectation_match = true`

You should get `60 / 60`.

### Check Individual Examples

Readers who want to inspect concrete cases can use the case IDs.

Examples:

- `SAN-001` shows a successful plain integer case
- `SAN-007` shows a safe rejection for an unsupported thousands separator
- `SAN-021` shows a no-number case
- `SAN-050` shows a mixed case where one numeric expression is unsupported and another is valid, but the case still ends as safe rejection because clarification is required

These cases can be checked in both `sanitization_results.json` and `sanitization_results.csv`.

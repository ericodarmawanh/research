# LLM Contextual Resolution Under Universal Masking

## What This File Is For

This file is intended for readers of the paper who want to:

- understand what was evaluated
- understand what the dataset contains
- understand what the result files contain
- understand what each category, style, and action means
- understand how the reported metrics were defined and computed
- verify the reported numbers directly from the supplementary files

## The Four Supplementary Files In This Folder

This supplementary package contains exactly four files:

- `evaluation_dataset.json`: the evaluation dataset
- `evaluation_results.json`: the full structured evaluation results
- `evaluation_results.csv`: the tabular export of the same evaluation results
- `evaluation_report.md`: this explanatory guide

## What Was Evaluated

This evaluation tests whether an LLM can still resolve contextual meaning after person names and addresses have been replaced by a universal neutral placeholder of the form `[token-ent: xxx]`.

The LLM only receives dialogue text that already contains placeholders such as `[token-ent: 101]`, and it must infer, from context alone, whether each placeholder refers to:

- a `name`
- an `address`
- or an ambiguous value that still requires clarification

The LLM must then produce a structured decision that contains:

- `resolved_entities`
- `action`

## What Universal Masking (Neutral Placeholder) Means Here

In this benchmark, **universal masking (neutral placeholder)** means that the original entity type is not exposed to the LLM.

For example, the model does not receive labels such as:

- `PERSON`
- `LOCATION`
- `NAME`
- `ADDRESS`

Instead, the model receives only a neutral placeholder:

- `[token-ent: 101]`
- `[token-ent: 302]`
- `[token-ent: 909]`

This design forces the model to use dialogue context, lexical cues, and task context rather than explicit entity-type labels.

## What The Dataset File Contains

Each object in `evaluation_dataset.json` is one evaluation case.

Main fields:

- `id`: unique case ID
- `category`: scenario type
- `style`: language style (`formal` or `noisy`)
- `prompt`: the dialogue text shown to the LLM
- `expected_output`: the gold structured decision for that case

Inside `expected_output`, there are two parts:

- `resolved_entities`
- `action`

Inside `resolved_entities`, there are always two fixed slots:

- `name`
- `address`

Each slot contains either:

- a placeholder such as `[token-ent: 101]`
- or `null`

## What The Results File Contains

Each object in `evaluation_results.json` is one evaluated case.

Main fields:

- `id`: unique case ID
- `category`: scenario type
- `style`: language style
- `prompt`: the input dialogue shown to the model
- `expected_output`: the gold structured decision
- `actual_output`: the structured decision produced by the model after parsing
- `passed`: whether `actual_output` exactly matches `expected_output`
- `parse_error`: parsing failure information, if any
- `raw_response_text`: the original raw model output before parsing

The top-level `summary` object contains the aggregate metrics for the full 90-case run.

## What The CSV File Contains

The CSV file is a row-wise tabular export of the same case-level evaluation results.

Columns:

- `id`
- `category`
- `style`
- `expected_action`
- `actual_action`
- `expected_name`
- `actual_name`
- `expected_address`
- `actual_address`
- `passed`
- `parse_error`

The CSV is useful for filtering failures, grouping by category, and checking formal versus noisy performance quickly.

## Output Structure Used In The Benchmark

For each case, the benchmark expects a structured output with this logical form:

```json
{
  "resolved_entities": {
    "name": null,
    "address": null
  },
  "action": "needs_confirmation"
}
```

### Meaning of `resolved_entities`

- `name`: the placeholder assigned to the name slot, if the context clearly supports a name interpretation
- `address`: the placeholder assigned to the address slot, if the context clearly supports an address interpretation
- `null`: used when the slot should remain empty

### Meaning of `action`

The action field encodes the downstream interpretation the system should take.

Allowed actions in this benchmark:

- `search_account_by_name`
- `search_cust_id_by_name`
- `search_cust_id_by_address`
- `search_cust_id_by_name_and_address`
- `needs_confirmation`

## What Each Action Means

### `search_account_by_name`

Use this action in transfer scenarios where the placeholder clearly refers to the account holder name.

### `search_cust_id_by_name`

Use this action in electricity-payment scenarios where the placeholder clearly refers to the customer or owner name.

### `search_cust_id_by_address`

Use this action in electricity-payment scenarios where the placeholder clearly refers to the house or service address.

### `search_cust_id_by_name_and_address`

Use this action when the dialogue clearly provides both a customer name and an address.

### `needs_confirmation`

Use this action when the context is still ambiguous and the system should ask for clarification instead of guessing.

This is the safe fallback action for unresolved ambiguity.

## What `style` Means

The dataset has two language styles.

### `formal`

This style uses relatively standard and explicit language.

Example pattern:

- `User: "Saya ingin membayar tagihan listrik atas nama [token-ent: 201]."`

### `noisy`

This style uses abbreviated, colloquial, or informal language.

Example pattern:

- `User: "byr pln pmlk [token-ent: 206] donk"`

The `noisy` label does not mean the text is incorrect.

It simply means the wording is less formal and may contain shortened expressions.

## What Each Category Means

There are 9 categories, each with 10 cases, for a total of 90 cases.

| Category | Meaning | Cases |
| --- | --- | ---: |
| `transfer_instruction_name` | direct transfer instruction where the masked entity is the recipient name | 10 |
| `electric_payment_name` | direct electricity-payment request where the masked entity is the customer name | 10 |
| `electric_payment_address` | direct electricity-payment request where the masked entity is the service address | 10 |
| `electric_payment_name_address` | electricity-payment request where both masked name and masked address are present | 10 |
| `short_reply_transfer_name` | short answer after the bot asks for the transfer-recipient name | 10 |
| `short_reply_electric_name` | short answer after the bot asks for the electricity-customer name | 10 |
| `short_reply_electric_address` | short answer after the bot asks for the electricity-service address | 10 |
| `short_reply_electric_ambiguous_1ent` | short reply with one placeholder where ambiguity must sometimes be preserved unless lexical cues are explicit | 10 |
| `short_reply_electric_ambiguous_2ent` | short reply with two placeholders where order alone must not be used to assign name and address roles | 10 |
| **Total** |  | **90** |

## Why The Ambiguous Categories Matter

The two ambiguous categories are important because they test whether the model stays conservative when the placeholder role is not justified by context.

### `short_reply_electric_ambiguous_1ent`

These cases ask whether a single placeholder should be interpreted as a customer name, an address, or left unresolved.

Within this category:

- 6 cases are intentionally ambiguous and should end in `needs_confirmation`
- 4 cases are clear because additional lexical cues explicitly identify the placeholder as `name` or `address`

### `short_reply_electric_ambiguous_2ent`

These cases ask whether two placeholders should be assigned to `name` and `address`.

Within this category:

- 6 cases are intentionally ambiguous and should remain unresolved
- 4 cases are clear because lexical cues explicitly identify which placeholder is the name and which is the address

These two categories create the 12 ambiguous cases used in the ambiguity-related metrics.

## Dataset Composition

The dataset contains:

- 90 total cases
- 45 formal cases
- 45 noisy cases
- 12 ambiguous cases
- 78 clear cases

The 12 ambiguous cases come from:

- 6 ambiguous cases in `short_reply_electric_ambiguous_1ent`
- 6 ambiguous cases in `short_reply_electric_ambiguous_2ent`

The other 78 cases are clear cases.

## Metric Definitions

### 1. Exact Match Rate

Question:

Did the full structured output exactly match the gold output for the case?

Formula:

`Exact Match Rate = exact_match_cases / total_cases * 100%`

For this run:

- `87 / 90 = 96.67%`

### 2. Action Accuracy

Question:

Did the model choose the correct `action` label?

Formula:

`Action Accuracy = correct_action_cases / total_cases * 100%`

For this run:

- `87 / 90 = 96.67%`

### 3. Name Slot Accuracy

Question:

Did the model assign the correct value to the `name` slot?

Formula:

`Name Slot Accuracy = correct_name_slot_cases / total_cases * 100%`

For this run:

- `87 / 90 = 96.67%`

### 4. Address Slot Accuracy

Question:

Did the model assign the correct value to the `address` slot?

Formula:

`Address Slot Accuracy = correct_address_slot_cases / total_cases * 100%`

For this run:

- `90 / 90 = 100.00%`

### 5. Ambiguity Handling Rate

Question:

Among cases that were truly ambiguous, how often did the model correctly remain conservative and output `needs_confirmation`?

Formula:

`Ambiguity Handling Rate = ambiguity_handled_cases / ambiguous_cases * 100%`

For this run:

- `12 / 12 = 100.00%`

### 6. False Resolution Rate

Question:

Among truly ambiguous cases, how often did the model incorrectly force a specific resolution instead of asking for clarification?

Formula:

`False Resolution Rate = false_resolution_cases / ambiguous_cases * 100%`

For this run:

- `0 / 12 = 0.00%`

### 7. False Clarification Rate

Question:

Among cases that were actually clear, how often did the model become unnecessarily conservative and output `needs_confirmation`?

Formula:

`False Clarification Rate = false_clarification_cases / clear_cases * 100%`

For this run:

- `3 / 78 = 3.85%`

### 8. Formal Exact Match Rate

Question:

What was the exact-match performance on the formal subset only?

Formula:

`Formal Exact Match Rate = formal_exact_match_cases / total_formal_cases * 100%`

For this run:

- `44 / 45 = 97.78%`

### 9. Noisy Exact Match Rate

Question:

What was the exact-match performance on the noisy subset only?

Formula:

`Noisy Exact Match Rate = noisy_exact_match_cases / total_noisy_cases * 100%`

For this run:

- `43 / 45 = 95.56%`

### 10. Parse Failure Count

Question:

How many model outputs could not be parsed into the expected structured format?

Formula:

`Parse Failure Count = number of cases with parsing failure`

For this run:

- `0`

## Raw Counts Used In The Final Results

| Item | Raw count | Value |
| --- | ---: | ---: |
| Total cases | 90 | 100.00% |
| Exact match | 87 / 90 | 96.67% |
| Correct action | 87 / 90 | 96.67% |
| Correct `name` slot | 87 / 90 | 96.67% |
| Correct `address` slot | 90 / 90 | 100.00% |
| Ambiguity handled correctly | 12 / 12 | 100.00% |
| False resolution | 0 / 12 | 0.00% |
| False clarification | 3 / 78 | 3.85% |
| Formal exact match | 44 / 45 | 97.78% |
| Noisy exact match | 43 / 45 | 95.56% |
| Parse failures | 0 / 90 | 0.00% |

## Why The Denominators Are Different

The denominators differ because the metrics answer different questions.

- exact match rate uses all 90 cases
- action accuracy uses all 90 cases
- slot accuracies use all 90 cases
- ambiguity handling rate uses only the 12 ambiguous cases
- false resolution rate uses only the 12 ambiguous cases
- false clarification rate uses only the 78 clear cases
- formal exact match rate uses only the 45 formal cases
- noisy exact match rate uses only the 45 noisy cases

This is why the report shows:

- `87 / 90`
- `12 / 12`
- `0 / 12`
- `3 / 78`
- `44 / 45`
- `43 / 45`

These are not inconsistent denominators.

They are metric-specific denominators.

## Category-Level Results

| Category | Cases | Exact Match | Action Accuracy |
| --- | ---: | ---: | ---: |
| `electric_payment_address` | 10 | 100.00% | 100.00% |
| `electric_payment_name` | 10 | 90.00% | 90.00% |
| `electric_payment_name_address` | 10 | 90.00% | 90.00% |
| `short_reply_electric_address` | 10 | 100.00% | 100.00% |
| `short_reply_electric_ambiguous_1ent` | 10 | 100.00% | 100.00% |
| `short_reply_electric_ambiguous_2ent` | 10 | 100.00% | 100.00% |
| `short_reply_electric_name` | 10 | 100.00% | 100.00% |
| `short_reply_transfer_name` | 10 | 90.00% | 90.00% |
| `transfer_instruction_name` | 10 | 100.00% | 100.00% |

## What The Three Observed Errors Look Like

This run has three failed cases.

### 1. `CAT2_NOISY_3`

- category: `electric_payment_name`
- style: `noisy`
- expected action: `search_cust_id_by_name`
- actual action: `needs_confirmation`

This is a false clarification case.

### 2. `CAT4_NOISY_3`

- category: `electric_payment_name_address`
- style: `noisy`
- expected action: `search_cust_id_by_name_and_address`
- actual action: `needs_confirmation`

This is also a false clarification case.

### 3. `CAT5_FORMAL_5`

- category: `short_reply_transfer_name`
- style: `formal`
- expected action: `search_account_by_name`
- actual action: `needs_confirmation`

This is also a false clarification case.

Because all three errors are conservative failures rather than over-confident failures, the benchmark records:

- `false_resolution_rate = 0.00%`
- `false_clarification_rate = 3.85%`

## Simple Examples

### Example of Direct Name Resolution

- category: `transfer_instruction_name`
- prompt: `User: "Tolong transfer dana ke rekening [token-ent: 101] sekarang."`
- expected output: `name = [token-ent: 101]`, `address = null`, `action = search_account_by_name`

### Example of Direct Address Resolution

- category: `electric_payment_address`
- prompt: `User: "Mohon lunasi tagihan PLN untuk rumah di [token-ent: 301]."`
- expected output: `name = null`, `address = [token-ent: 301]`, `action = search_cust_id_by_address`

### Example of Clear Name-and-Address Resolution

- category: `electric_payment_name_address`
- prompt: `User: "Bayarkan listrik untuk Bapak [token-ent: 401] yang berlokasi di [token-ent: 402]."`
- expected output: `name = [token-ent: 401]`, `address = [token-ent: 402]`, `action = search_cust_id_by_name_and_address`

### Example of Ambiguous Case That Must Stay Unresolved

- category: `short_reply_electric_ambiguous_1ent`
- prompt: `Bot: "Silakan masukkan nama pemilik atau alamat rumah untuk tagihan listriknya." User: "[token-ent: 801]."`
- expected output: `name = null`, `address = null`, `action = needs_confirmation`

### Example of Ambiguous Category With Clear Lexical Cue

- category: `short_reply_electric_ambiguous_2ent`
- prompt: `Bot: "Silakan sebutkan nama pemilik atau alamat untuk pembayaran PLN." User: "Nama [token-ent: 907] alamat [token-ent: 908]."`
- expected output: `name = [token-ent: 907]`, `address = [token-ent: 908]`, `action = search_cust_id_by_name_and_address`

## How To Recheck The Numbers From The Supplementary Files

A reader can verify the reported results directly from the four supplementary files.

### Check Dataset Size

Open `evaluation_dataset.json`.

Count the number of objects.

You should get:

- `90` total cases

### Check Style Counts

In `evaluation_dataset.json`, count:

- rows with `style = formal`
- rows with `style = noisy`

You should get:

- `45 formal`
- `45 noisy`

### Check The Top-Level Summary

Open `evaluation_results.json` and read the `summary` object.

You should see:

- `total_cases = 90`
- `exact_match_rate = 96.66666666666667`
- `action_accuracy = 96.66666666666667`
- `name_accuracy = 96.66666666666667`
- `address_accuracy = 100.0`
- `ambiguity_handling_rate = 100.0`
- `false_resolution_rate = 0.0`
- `false_clarification_rate = 3.8461538461538463`
- `formal_exact_match_rate = 97.77777777777777`
- `noisy_exact_match_rate = 95.55555555555556`
- `parse_failure_count = 0`

### Recompute Exact Match Rate

In `evaluation_results.json` or `evaluation_results.csv`:

- count all cases with `passed = true`
- divide by total cases

You should get:

- `87 / 90`

### Recompute Action Accuracy

In `evaluation_results.csv`:

- compare `expected_action` and `actual_action`
- count the matches
- divide by total rows

You should get:

- `87 / 90`

### Recompute Name Slot Accuracy

In `evaluation_results.csv`:

- compare `expected_name` and `actual_name`
- count the matches across all rows

You should get:

- `87 / 90`

### Recompute Address Slot Accuracy

In `evaluation_results.csv`:

- compare `expected_address` and `actual_address`
- count the matches across all rows

You should get:

- `90 / 90`

### Recompute Ambiguity Metrics

Use the ambiguous cases from the two ambiguous categories.

These are the cases whose gold output expects `action = needs_confirmation`.

Count them in `evaluation_dataset.json` or `evaluation_results.json`.

You should get:

- `12 ambiguous cases`

Then:

- ambiguity handling rate: count how many of those 12 still have `actual_output.action = needs_confirmation`
- false resolution rate: count how many of those 12 were incorrectly assigned a non-`needs_confirmation` action

You should get:

- `12 / 12` for ambiguity handling
- `0 / 12` for false resolution

### Recompute False Clarification Rate

A false clarification is a case that was actually clear, but the model still returned `needs_confirmation`.

You can verify this by:

- taking all clear cases, which total `78`
- counting the clear cases where `actual_action = needs_confirmation`

You should get:

- `3 / 78`

The three such cases are:

- `CAT2_NOISY_3`
- `CAT4_NOISY_3`
- `CAT5_FORMAL_5`

### Recompute Formal And Noisy Exact Match Rates

In `evaluation_results.csv`:

- filter rows where `style = formal`
- count `passed = true`
- divide by 45

You should get:

- `44 / 45 = 97.78%`

Then:

- filter rows where `style = noisy`
- count `passed = true`
- divide by 45

You should get:

- `43 / 45 = 95.56%`

## Notes On Reading The Files Together

A simple reading order for a paper reader is:

1. `evaluation_report.md`
2. `evaluation_dataset.json`
3. `evaluation_results.json`
4. `evaluation_results.csv`

The dataset file explains what each case asks the model to do.

The JSON result file shows the exact structured predictions and aggregate metrics.

The CSV file makes it easier to inspect failures, compare actions, and filter by category or style.

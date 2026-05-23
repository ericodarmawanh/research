# Labeling Scheme

## Overview

The fine-tuning dataset uses a BIO token-labeling scheme for two target entity types:

- `PER`: person name
- `ADDR`: address or address detail

The full label set is:

- `O`
- `B-PER`
- `I-PER`
- `B-ADDR`
- `I-ADDR`

This label set matches the label space used by the training code.

## Meaning of each label

| Label | Meaning |
| --- | --- |
| `O` | The token is not part of a target entity |
| `B-PER` | The token is the first token of a person entity |
| `I-PER` | The token is a continuation token of the same person entity |
| `B-ADDR` | The token is the first token of an address entity |
| `I-ADDR` | The token is a continuation token of the same address entity |

## Why BIO is used

BIO labeling is useful because the model must learn both:

1. the **entity type** (`PER` or `ADDR`)
2. the **entity boundary**, meaning where the entity starts and where it ends

Without a boundary-aware scheme, the model could identify the correct type but still select a span that is too short or too long.

## Worked examples

### Example 1: person name

Text:

`Transfer ke Asep Nainggolan.`

Illustrative token labels:

| Token | Label |
| --- | --- |
| `Transfer` | `O` |
| `ke` | `O` |
| `Asep` | `B-PER` |
| `Nainggolan` | `I-PER` |

Interpretation:

- `Asep` starts the person entity, so it receives `B-PER`
- `Nainggolan` continues the same person entity, so it receives `I-PER`

### Example 2: address

Text:

`Bayarkan listrik rumah saya yang di jalan Cideng Timur no. 28.`

Illustrative token labels:

| Token | Label |
| --- | --- |
| `Bayarkan` | `O` |
| `listrik` | `O` |
| `rumah` | `O` |
| `saya` | `O` |
| `yang` | `O` |
| `di` | `O` |
| `jalan` | `B-ADDR` |
| `Cideng` | `I-ADDR` |
| `Timur` | `I-ADDR` |
| `no.` | `I-ADDR` |
| `28` | `I-ADDR` |

Interpretation:

- `jalan` starts the address span, so it receives `B-ADDR`
- the following address tokens continue the same entity and therefore receive `I-ADDR`

## Boundary behavior

BIO labeling makes span boundaries explicit.

For example:

- `Pak Budi` in the surface text does **not** always mean the gold span is `Pak Budi`
- depending on the annotation policy, the gold person entity may be only `Budi`

This is important because the benchmark and the training data both care about **exact span selection**, not only coarse entity type assignment.

## Relation to the JSONL format

The JSONL files store entity spans as character offsets:

- `start`
- `end`
- `label`
- `text`

During training, the tokenizer aligns those span annotations to token-level BIO labels.

So:

1. the dataset is authored as span annotations in JSONL
2. the training pipeline converts those spans into token-level BIO labels
3. the model is trained to predict BIO labels token by token

## Summary

Readers who want to reproduce the training setup should interpret the fine-tuning dataset as:

- span-annotated JSONL examples at the file level
- converted into a five-label BIO token-classification problem during preprocessing

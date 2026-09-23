# ParchiVisa — Official Question Schema Reference

**Version:** 1.0.0  
**Date:** 2026-06-06  
**Applies to:** All country `questions.json` files under `student/` and any future visa type directories  
**Status:** Active — use this document as the authoritative reference before adding or cleaning any question object

---

## Table of Contents

1. [Purpose and Scope](#1-purpose-and-scope)
2. [Canonical Field Reference](#2-canonical-field-reference)
3. [Required vs Optional Fields](#3-required-vs-optional-fields)
4. [Current-to-Target Field Name Migration](#4-current-to-target-field-name-migration)
5. [Input Type Catalog](#5-input-type-catalog)
6. [Options Format](#6-options-format)
7. [Validation Rules by Input Type](#7-validation-rules-by-input-type)
8. [Scoring Fields — Why Internal Values Matter](#8-scoring-fields--why-internal-values-matter)
9. [Date Standard](#9-date-standard)
10. [Currency Standard](#10-currency-standard)
11. [Conditional Logic — show_if](#11-conditional-logic--show_if)
12. [Blocker-Related Fields](#12-blocker-related-fields)
13. [Source and Rule Reference Fields](#13-source-and-rule-reference-fields)
14. [Good Question Examples](#14-good-question-examples)
15. [Bad Question Examples](#15-bad-question-examples)
16. [Dataset Cleanup Rules for New Countries](#16-dataset-cleanup-rules-for-new-countries)

---

## 1. Purpose and Scope

This document defines the canonical structure every question object in ParchiVisa must follow. It is the single source of truth for:

- What fields are required and what fields are optional
- The exact meaning and expected format of every field
- Which `input_type` values are valid and what each implies
- How options, validation, conditional logic, and scoring must be structured
- How dates and currency amounts must be stored
- What "good" and "bad" question data looks like

This is a **data schema document**. It does not describe application code, frontend rendering behaviour, or immigration law. Any field that involves legal interpretation must be verified by a human expert and confirmed with `needs_rule_confirmation: false` before the dataset is used in production.

### What this document does not cover

- Backend scoring engine implementation
- Frontend component rendering rules
- Database migration scripts
- Immigration law or policy interpretation

---

## 2. Canonical Field Reference

The table below defines every field a question object may contain. Fields marked **Required** must be present on every question. Fields marked **Conditional** must be present when a specific condition applies. Fields marked **Optional** may be omitted when not applicable.

| Field | Type | Status | Description |
|---|---|---|---|
| `id` | `string` | **Required** | Stable, unique question identifier. Format: `{country_code}_{descriptor}`. Example: `aus_has_coe`. Must never change once assigned. |
| `country` | `string` | **Required** | Display name of the destination country. Examples: `"Australia"`, `"Canada"`, `"UK"`, `"USA"`. |
| `visa_route` | `string` | **Required** | Full human-readable visa route name. Example: `"Student Visa (Subclass 500)"`. |
| `section` | `string` | **Required** | Logical grouping within the questionnaire. Examples: `"Financial"`, `"Immigration History"`, `"CAS / Sponsorship"`. Used for display grouping only. |
| `question` | `string` | **Required** | The question text shown to the user. Must be a complete, plain-language sentence ending with a question mark. No abbreviations. No legal jargon unless unavoidable. |
| `help_text` | `string` | **Required** | Explanatory text shown alongside the question. Should explain why the question is asked, what a failure means, and what the user should do. Must not duplicate `question`. Must not give legal advice. |
| `input_type` | `string` | **Required** | The UI control type. Must be one of the 12 supported values listed in Section 5. |
| `required` | `boolean` | **Required** | Whether an answer is required before the questionnaire can advance. Almost always `true`. Use `false` only for genuinely optional supplementary questions. |
| `options` | `array` | **Conditional** | Required when `input_type` is `yes_no`, `yes_no_unknown`, `single_choice`, or `multi_choice`. Must be an array of `{ "label": string, "value": string }` objects. See Section 6. |
| `validation` | `object` | **Required** | Defines pass and trigger logic. Contains `pass_if` and `trigger_if` sub-objects. See Section 7. |
| `show_if` | `object \| null` | **Required** | Conditional display rule. `null` means always show. When present, must be a structured object. See Section 11. |
| `mapped_rule` | `string` | **Required** | Human-readable name of the immigration rule this question tests. Must match the `rule_name` field in the corresponding `rules.json`. |
| `mapped_rule_id` | `string \| null` | **Required** | Stable ID of the rule in `rules.json`. `null` only when a confident match cannot be made; in that case `needs_rule_confirmation` must be `true`. |
| `scoring_key` | `string` | **Required** | The scoring category this question contributes to. Must match a `category_id` in `scoring.json`. Examples: `"financial_legitimacy"`, `"immigration_history"`. |
| `risk_category` | `string` | **Required** | The risk level if this question fails. Must be one of: `"Critical"`, `"High"`, `"Medium"`, `"Low"`. Case-sensitive. |
| `blocker_possible` | `boolean` | **Required** | Whether a failing answer on this question can produce a hard blocker. `true` for questions mapped to hard requirements. `false` for advisory and awareness questions. |
| `normalized_answer_format` | `string` | **Required** | The format the answer will be stored in after normalisation. Examples: `"boolean"`, `"YYYY-MM-DD"`, `"numeric_pkr"`, `"string_lowercase"`, `"array_of_strings"`. See Sections 9 and 10. |
| `error_message` | `string` | **Required** | The message shown to the user when their answer triggers a warning or blocker. Must be direct and actionable. Must not be a generic placeholder. |
| `source_url` | `string` | **Conditional** | Direct URL to the official government or institutional source for this question's legal basis. Required when `source_resolution_status` is `"direct_url"`. |
| `source_ids` | `array` | **Conditional** | Array of source reference IDs that resolve via a `sources.json` lookup file. Required when `source_resolution_status` is `"unresolved"` and a lookup file exists. Not required if `source_url` is present. |
| `source_resolution_status` | `string` | **Required** | Must be one of: `"direct_url"` (a usable URL is present), `"unresolved"` (source IDs exist but no lookup file yet), `"missing"` (no source exists at all — flag for review). |
| `last_verified` | `string` | **Required** | ISO date `YYYY-MM-DD` when a human last confirmed this question's legal accuracy. Must be updated whenever question content is reviewed. |
| `needs_rule_confirmation` | `boolean` | **Conditional** | `true` when `mapped_rule_id` is `null` because no confident rule match could be made. Omit entirely (or set `false`) when rule linkage is confirmed. |

### Fields present in current JSON files that do not appear above

The following fields exist in the current `questions.json` files. They predate this schema and will be migrated or renamed during question-level cleanup. Do not remove them until migration is complete.

| Current field | Maps to schema field | Notes |
|---|---|---|
| `question_id` | `id` | Rename during cleanup |
| `question_text` | `question` | Rename during cleanup |
| `user_help_text` | `help_text` | Rename during cleanup |
| `score_category` | `scoring_key` | Rename during cleanup |
| `risk_level` | `risk_category` | Rename during cleanup |
| `conditional_on` | `show_if` | Format also changes — see Section 11 |
| `pass_if` / `trigger_if` | `validation` | Merged into single `validation` object — see Section 7 |
| `tier` | informs `blocker_possible` | `"hard"` → `blocker_possible: true`; others → `false`. See Section 12 |
| `score_impact` | no direct equivalent yet | Retained as-is pending scoring model finalisation |

---

## 3. Required vs Optional Fields

### Always required (every question, no exceptions)

```
id, country, visa_route, section, question, help_text,
input_type, required, validation, show_if, mapped_rule,
mapped_rule_id, scoring_key, risk_category, blocker_possible,
normalized_answer_format, error_message,
source_resolution_status, last_verified
```

### Required when a condition applies

| Field | Required when |
|---|---|
| `options` | `input_type` is `yes_no`, `yes_no_unknown`, `single_choice`, or `multi_choice` |
| `source_url` | `source_resolution_status` is `"direct_url"` |
| `source_ids` | `source_resolution_status` is `"unresolved"` and a `sources.json` lookup file exists |
| `needs_rule_confirmation` | `mapped_rule_id` is `null` |

### Optional fields

| Field | Use when |
|---|---|
| `source_ids` | Multiple sources back one question and a lookup file exists |
| `needs_rule_confirmation: false` | Omit entirely — absence means confirmed |

---

## 4. Current-to-Target Field Name Migration

This table is the authoritative mapping for anyone performing question-level cleanup on existing JSON files. Do not rename fields in isolation — rename them as a batch using an automated script to avoid partial states.

| Current name | Target name | Change type |
|---|---|---|
| `question_id` | `id` | Rename only |
| `question_text` | `question` | Rename only |
| `user_help_text` | `help_text` | Rename only |
| `score_category` | `scoring_key` | Rename only |
| `risk_level` | `risk_category` | Rename only |
| `conditional_on` | `show_if` | Rename + format change (string → object) |
| `pass_if` + `trigger_if` | `validation` | Merge into object |
| `tier` | `blocker_possible` | Type change (string → boolean) |
| `options` (raw strings) | `options` (label/value objects) | Format change |
| _(absent)_ | `normalized_answer_format` | New field — add during cleanup |
| _(absent)_ | `error_message` | New field — add during cleanup |
| _(absent)_ | `source_resolution_status` | Already added during normalisation |

---

## 5. Input Type Catalog

### Supported values

Only the 12 values below are valid for `input_type`. Any other value is a data error.

---

#### `yes_no`

A binary question with exactly two options: yes or no.

- Options required: `[{ "label": "Yes", "value": "yes" }, { "label": "No", "value": "no" }]`
- Normalised answer: `"yes"` or `"no"` (string lowercase)
- Use when: the answer is genuinely binary with no ambiguity
- Do not use when: the applicant may legitimately not know, or "not applicable" is a meaningful option

---

#### `yes_no_unknown`

A three-way question for yes / no / uncertain. Replaces all current variants: `yes_no_unsure`, `yes_no_not_sure`, `yes_no_not_yet`.

- Options required: `[{ "label": "Yes", "value": "yes" }, { "label": "No", "value": "no" }, { "label": "Not sure", "value": "unknown" }]`
- Normalised answer: `"yes"`, `"no"`, or `"unknown"`
- Use when: the applicant might not yet know the answer, or uncertainty is a meaningful risk signal
- Note on cleanup: `yes_no_unsure` (options: yes/no/unsure), `yes_no_not_sure` (options: yes/no/not_sure), and `yes_no_not_yet` (options: yes/no/not_yet) all map to this type. The option **value** may differ per country until cleanup — do not change values without updating `validation` logic simultaneously.

---

#### `single_choice`

A single-selection question with three or more options. Replaces the current `dropdown` type.

- Options required: array of `{ "label": string, "value": string }` with 3 or more entries
- Normalised answer: the selected option's `value` (string lowercase, snake_case)
- Use when: the answer must be exactly one of a fixed set (e.g. fund source, course location)
- Do not use when: the user can select multiple answers — use `multi_choice` instead

---

#### `multi_choice`

A multi-selection question. Replaces the current `multi_select` type.

- Options required: array of `{ "label": string, "value": string }` with 2 or more entries
- Normalised answer: `array` of selected `value` strings
- Use when: the applicant may have multiple applicable answers (e.g. types of ties to home country, types of financial documents available)
- Always include a `"none"` option when "none of the above" is a meaningful and distinct answer
- Scoring and validation logic must reference option **values**, never display labels

---

#### `date`

A date input.

- Options: not required (empty array or omit)
- User-facing display format: `DD/MM/YYYY`
- Normalised stored format: `YYYY-MM-DD`
- See Section 9 for the full date standard
- Use when: a specific calendar date is needed (e.g. programme start date, passport expiry)

---

#### `number`

A numeric integer or decimal input. Use for counts, ages, durations.

- Options: not required
- Normalised answer: numeric (no currency symbol, no thousands separator)
- Use when: the value is a dimensionless number (age, months, year)
- Do not use for monetary amounts — use `currency_amount` instead

---

#### `currency_amount`

A monetary value. Currently some financial questions use `number` — those should be migrated to this type during cleanup.

- Options: not required
- User-facing display: may include currency symbol or code (e.g. `AUD 29,710`)
- Normalised stored format: plain numeric value — the currency is stored separately in the question's metadata or in `scoring.json` config
- Example: user sees `£ 13,000`, stored as `13000`
- See Section 10 for the full currency standard

---

#### `text`

A free-text input. Use sparingly — free text cannot be validated or scored reliably.

- Options: not required
- Normalised answer: trimmed string
- Use when: the answer is genuinely open-ended (e.g. list of prior refusal countries and reasons)
- Do not use when: the possible answers form a finite set — use `single_choice` or `multi_choice` instead
- Scoring impact must be minimal or zero for `text` questions because the answer cannot be programmatically evaluated

---

#### `email`

An email address input.

- Options: not required
- Normalised answer: lowercase trimmed string
- Validation: must match standard email format (`user@domain.tld`)
- Use when: contact or account email is needed (currently not used in student visa questions)

---

#### `phone`

A phone number input.

- Options: not required
- Normalised answer: E.164 format string (e.g. `+923001234567`)
- Use when: contact phone number is needed (currently not used in student visa questions)

---

#### `country`

A country selector. User picks a country from a standardised list.

- Options: populated from a shared country reference list, not hardcoded per question
- Normalised answer: ISO 3166-1 alpha-2 code (e.g. `"PK"`, `"AU"`, `"GB"`)
- Use when: the question asks which country (e.g. countries where the applicant has lived)

---

#### `file_upload`

A document upload input.

- Options: not required
- Normalised answer: file reference object or URL — format defined by the upload implementation
- Use when: the question asks the applicant to upload supporting evidence
- Note: file upload questions must have `required: false` unless the system can technically enforce the upload before advancing

---

### Current input_type values and their target equivalents

| Current value | Target value | Action required |
|---|---|---|
| `yes_no` | `yes_no` | No change |
| `yes_no_unsure` | `yes_no_unknown` | Rename; update option values to `unknown` |
| `yes_no_not_sure` | `yes_no_unknown` | Rename; update option values to `unknown` |
| `yes_no_not_yet` | `yes_no_unknown` | Rename; update option values to `unknown` |
| `yes_no_na` | `single_choice` | Rename; keep `not_applicable` option value |
| `dropdown` | `single_choice` | Rename only |
| `multi_select` | `multi_choice` | Rename only |
| `number` | `number` | No change (except financial fields → `currency_amount`) |
| `text` | `text` | No change |
| `date` | `date` | No change |

---

## 6. Options Format

### Target format

Every option in any `options` array must be an object with exactly two keys:

```json
{ "label": "Display text shown to the user", "value": "internal_value_used_by_scoring" }
```

- `label`: Plain language, sentence case. What the user reads on screen.
- `value`: Lowercase snake_case internal identifier. What scoring, validation, and conditional logic reference. **Never change a `value` once it is in production** — it will break all downstream logic referencing it.

### Current format (pre-cleanup)

Current `questions.json` files use raw string arrays:

```json
"options": ["yes", "no", "unsure"]
```

This is a known pre-schema pattern. Do not modify existing option arrays until the full question-level cleanup task is executed, because changing option values without simultaneously updating `pass_if`, `trigger_if`, and `conditional_on` references will break validation logic.

### Standard option sets

These sets are reused across all countries. Use exactly these values — do not invent synonyms.

**yes_no:**
```json
[
  { "label": "Yes", "value": "yes" },
  { "label": "No", "value": "no" }
]
```

**yes_no_unknown:**
```json
[
  { "label": "Yes", "value": "yes" },
  { "label": "No", "value": "no" },
  { "label": "Not sure", "value": "unknown" }
]
```

**yes_no with not_applicable:**
```json
[
  { "label": "Yes", "value": "yes" },
  { "label": "No", "value": "no" },
  { "label": "Not applicable", "value": "not_applicable" }
]
```

### Why scoring must use internal values, not display labels

Scoring and conditional logic reference option `value`s, not `label`s. If you use labels in logic:

- A label change (e.g. "Not sure" → "I don't know") silently breaks all scoring rules referencing that label
- Translations will be impossible without rewriting scoring logic
- A/B testing display text requires no data changes when logic uses values

**This is not optional.** Every `pass_if`, `trigger_if`, `show_if`, and scoring weight must reference the `value` key, never the `label`.

---

## 7. Validation Rules by Input Type

Each question's `validation` object contains two sub-objects: `pass_if` and `trigger_if`. These define what counts as a passing answer and what triggers a warning or blocker.

### Structure

```json
"validation": {
  "pass_if": { "type": "...", "value": "..." },
  "trigger_if": { "type": "...", "value": "..." }
}
```

### Supported validation types

| Type | Applies to | Meaning |
|---|---|---|
| `eq` | any | Answer equals `value` |
| `in` | any | Answer is contained in `value` (array) |
| `gte` | `number`, `currency_amount` | Answer is greater than or equal to `value` |
| `gt` | `number`, `currency_amount` | Answer is greater than `value` |
| `lte` | `number`, `currency_amount` | Answer is less than or equal to `value` |
| `lt` | `number`, `currency_amount` | Answer is less than `value` |
| `multi_has_any` | `multi_choice` | At least one selected value is in the `value` array |
| `multi_contains` | `multi_choice` | The selected values include `value` (string or array) |
| `computed_funds_ok` | `currency_amount` | Passes a computed financial threshold (defined in `scoring.json` config) |
| `computed_funds_short` | `currency_amount` | Fails the same computed threshold |
| `any` | any | Always passes — used for informational questions |
| `never` | any | Never triggers — used with informational questions where no answer is a blocker |

### Rules by input_type

**`yes_no`** — must use `eq` with `"yes"` or `"no"`, or `in` with an array:
```json
"pass_if":    { "type": "eq", "value": "yes" },
"trigger_if": { "type": "eq", "value": "no" }
```

**`yes_no_unknown`** — pass on `"yes"`, trigger on `"no"` and/or `"unknown"`:
```json
"pass_if":    { "type": "eq", "value": "yes" },
"trigger_if": { "type": "in", "value": ["no", "unknown"] }
```

**`single_choice`** — use `eq` or `in`:
```json
"pass_if":    { "type": "in", "value": ["london", "outside_london"] },
"trigger_if": { "type": "never" }
```

**`multi_choice`** — use `multi_has_any` for pass, `multi_contains` for trigger:
```json
"pass_if":    { "type": "multi_has_any", "value": ["property_or_assets", "family_ties"] },
"trigger_if": { "type": "multi_contains", "value": "none" }
```

**`number`** — use `gte`, `gt`, `lte`, `lt`:
```json
"pass_if":    { "type": "gte", "value": 18 },
"trigger_if": { "type": "lt", "value": 18 }
```

**`currency_amount`** — use computed types when the threshold comes from `scoring.json`, or `gte` when it is a fixed figure:
```json
"pass_if":    { "type": "computed_funds_ok" },
"trigger_if": { "type": "computed_funds_short" }
```

**`date`** — computed validation is preferred; for awareness questions:
```json
"pass_if":    { "type": "any" },
"trigger_if": { "type": "never" }
```

**`text`** — almost always informational; use `any` / `never`:
```json
"pass_if":    { "type": "any" },
"trigger_if": { "type": "never" }
```

**`file_upload`** — use `any` unless file presence can be technically verified:
```json
"pass_if":    { "type": "any" },
"trigger_if": { "type": "never" }
```

### Informational questions

Questions with `tier: "info"` in the current data are awareness checks. They should always have:
```json
"pass_if":    { "type": "any" },
"trigger_if": { "type": "never" }
```
These questions contribute to `score_impact` but never produce warnings or blockers.

---

## 8. Scoring Fields — Why Internal Values Matter

### Fields involved

| Field | Type | Description |
|---|---|---|
| `scoring_key` | `string` | The scoring category this question contributes to. Must match `category_id` in `scoring.json`. |
| `score_impact` | `integer` | Points contributed to the category when the question passes. Defined per question. |
| `risk_category` | `string` | The risk level of a failing answer: `"Critical"`, `"High"`, `"Medium"`, `"Low"`. |
| `blocker_possible` | `boolean` | Whether a failing answer can produce a hard blocker. See Section 12. |

### Why scoring must use internal values

The scoring engine evaluates answers by reading the `value` stored in the normalised answer. If options use display labels as values (e.g. `"Yes, I have proof"` instead of `"yes"`), the scoring engine must be updated every time display text changes. This makes the data fragile.

**Rule:** Every scoring weight, threshold, and condition must reference the internal `value` of an option, not its `label`. This applies to `pass_if`, `trigger_if`, `multi_has_any`, `multi_contains`, and any computed logic in `scoring.json`.

### scoring_key values in use

Each country uses its own set of `scoring_key` values that match its `scoring.json`. When adding questions for a new country, verify each `scoring_key` value against the country's `scoring.json` `scoring_categories` list before committing.

---

## 9. Date Standard

| Context | Format | Example |
|---|---|---|
| User-facing display | `DD/MM/YYYY` | `29/05/2026` |
| Stored / normalised value | `YYYY-MM-DD` (ISO 8601) | `2026-05-29` |
| `last_verified` field | `YYYY-MM-DD` | `2026-05-29` |
| `generated_at` in index.json | `YYYY-MM-DD` | `2026-06-06` |

### Rules

- Never store a date as `DD/MM/YYYY` in JSON — this format is for display only
- Never store a date as a Unix timestamp in question data
- If a date cannot be determined (e.g. source material does not give a date), use the string `"unknown"` — do not fabricate a date
- `last_verified` must be updated every time question content is reviewed by a human, even if no changes are made — it is a recency signal, not a change log

---

## 10. Currency Standard

### Principles

Currency questions (funds available, tuition fees, visa application charges) must separate the **display representation** from the **stored value**.

| Context | Format | Example |
|---|---|---|
| User-facing display | May include currency symbol or code | `AUD 29,710` or `£13,761` |
| Stored / normalised value | Plain numeric, no symbol, no separator | `29710` |
| Currency code | Stored separately, never inside the numeric value | `"currency": "AUD"` |

### Rules

- Do not store `"29,710"` (string with comma) — store `29710` (integer)
- Do not store `"AUD 29710"` (string with currency code) — the currency belongs in metadata, not in the answer value
- Conversion rates between PKR and foreign currencies are dynamic. Do not hardcode a rate inside a question. Rates belong in `scoring.json` config or a separate rates file.
- When a threshold is government-set (e.g. AUD 29,710 living cost), document its source URL and `last_verified` date. These figures change and must be reviewed before each new applicant cohort.

---

## 11. Conditional Logic — show_if

### Purpose

`show_if` controls whether a question is displayed to the user. A question with `show_if: null` is always shown. A question with a `show_if` object is shown only when the condition evaluates to `true`.

### Target format

```json
"show_if": {
  "question_id": "aus_edu_gap_over_1yr",
  "operator": "eq",
  "value": "yes"
}
```

Fields:

| Key | Type | Description |
|---|---|---|
| `question_id` | `string` | The `id` of the question whose answer is being tested |
| `operator` | `string` | The comparison operator: `eq`, `in`, `neq`, `not_in` |
| `value` | `string \| array` | The value (or array of values) to compare against |

### Current format (pre-cleanup)

Current `conditional_on` is a freeform string:
```
"conditional_on": "aus_edu_gap_over_1yr = yes"
```

Some entries use descriptive text rather than question IDs:
```
"conditional_on": "packaged courses only"
"conditional_on": "including dependants"
```

These freeform strings cannot be evaluated programmatically. During cleanup, each one must be replaced with a structured `show_if` object or set to `null` if the condition is not machine-evaluable.

### When show_if is null

Set `show_if: null` when the question should always appear regardless of any prior answer.

### Nested conditions

If a question depends on multiple prior answers, use a top-level `and` or `or` wrapper:

```json
"show_if": {
  "operator": "and",
  "conditions": [
    { "question_id": "aus_has_dependants", "operator": "eq", "value": "yes" },
    { "question_id": "aus_is_main_applicant", "operator": "eq", "value": "yes" }
  ]
}
```

Do not nest more than two levels deep. If logic is more complex, split into separate questions.

---

## 12. Blocker-Related Fields

### Current tier system

Current `questions.json` files use a `tier` field with four values:

| Tier | Meaning |
|---|---|
| `hard` | A failing answer is a hard blocker — the application cannot proceed |
| `high_risk` | A failing answer is a major risk flag, but not a hard blocker |
| `soft` | A failing answer is a soft warning — advisory only |
| `info` | The question is informational — no pass/fail consequence |

### Target blocker field

The `tier` field is retained during the current phase. The `blocker_possible` boolean is added as an additional field:

| tier | blocker_possible |
|---|---|
| `hard` | `true` |
| `high_risk` | `false` |
| `soft` | `false` |
| `info` | `false` |

`blocker_possible: true` means that if `trigger_if` evaluates to `true`, the question's `error_message` is surfaced as a hard stop and the scoring engine records a hard blocker event.

### error_message for blockers

Questions with `blocker_possible: true` must have an `error_message` that is:

- Specific to the exact failure condition
- Actionable — it tells the user what to do, not just what went wrong
- Free of legal advice language
- No longer than two sentences

**Good:**
```
"error_message": "You cannot lodge a Subclass 500 without a valid CoE from a CRICOS-registered provider. Obtain your CoE from the institution before continuing."
```

**Bad:**
```
"error_message": "This field is required."
```

---

## 13. Source and Rule Reference Fields

### Source fields

| Field | Purpose |
|---|---|
| `source_url` | Direct link to the official government or institutional page confirming the rule. Preferred over `source_ids`. |
| `source_ids` | Array of IDs referencing a `sources.json` lookup file. Currently used by USA questions; no lookup file exists yet. |
| `source_resolution_status` | Machine-readable indicator of source health. |

### source_resolution_status values

| Value | Meaning | Action required |
|---|---|---|
| `"direct_url"` | A working `source_url` is present | None — this is the target state |
| `"unresolved"` | `source_ids` are present but no lookup file exists | Create `sources.json` to resolve; or add `source_url` directly |
| `"missing"` | No source reference exists at all | Research and add `source_url` before marking content as production-ready |

### Rule reference fields

| Field | Purpose |
|---|---|
| `mapped_rule` | Human-readable name of the immigration rule. Must match `rule_name` in `rules.json`. |
| `mapped_rule_id` | Stable ID of the rule. Must match `rule_id` in `rules.json`. |
| `needs_rule_confirmation` | Set to `true` when the rule link is uncertain. |

### When to set needs_rule_confirmation: true

Set `needs_rule_confirmation: true` and `mapped_rule_id: null` when:

- No rule in `rules.json` clearly and specifically corresponds to this question
- The closest rule match is ambiguous between two or more rules
- The immigration basis for the question is uncertain and has not been verified by a legal/migration expert

Never invent a `mapped_rule_id` to avoid leaving this field empty. An explicit `null` with `needs_rule_confirmation: true` is far more honest and safer than a confident-looking but wrong rule link.

---

## 14. Good Question Examples

### Example A — Hard blocker question (yes_no)

```json
{
  "id": "aus_has_coe",
  "country": "Australia",
  "visa_route": "Student Visa (Subclass 500)",
  "section": "Enrolment",
  "question": "Do you already hold a valid Confirmation of Enrolment (CoE) from your Australian institution?",
  "help_text": "You cannot lodge a Subclass 500 without a valid CoE from a CRICOS-registered provider. An offer letter is not a CoE — the CoE is issued after you accept and pay the deposit.",
  "input_type": "yes_no",
  "required": true,
  "options": [
    { "label": "Yes", "value": "yes" },
    { "label": "No", "value": "no" }
  ],
  "validation": {
    "pass_if":    { "type": "eq", "value": "yes" },
    "trigger_if": { "type": "eq", "value": "no" }
  },
  "show_if": null,
  "mapped_rule": "Confirmation of Enrolment (CoE) Required",
  "mapped_rule_id": "aus_student_coe_required",
  "scoring_key": "enrolment_documents",
  "score_impact": 10,
  "risk_category": "Critical",
  "blocker_possible": true,
  "normalized_answer_format": "string_lowercase",
  "error_message": "A valid CoE from a CRICOS-registered provider is mandatory. Your application cannot proceed without one.",
  "source_url": "https://immi.homeaffairs.gov.au/visas/getting-a-visa/visa-listing/student-500",
  "source_resolution_status": "direct_url",
  "last_verified": "2026-05-29"
}
```

**Why this is good:**
- `id` is stable, namespaced, descriptive
- `question` is plain language and ends with a question mark
- `help_text` explains the CoE vs offer letter distinction — actionable
- `options` use label/value objects with clean internal values
- `validation` uses `eq` — unambiguous
- `show_if` is explicitly `null`
- `mapped_rule_id` links to a confirmed rule
- `blocker_possible: true` matches `trigger_if` logic
- `error_message` is specific and tells the user what to do
- `source_url` is a direct government URL
- `last_verified` is present

---

### Example B — Multi-choice with none sentinel

```json
{
  "id": "can_home_country_ties",
  "country": "Canada",
  "visa_route": "Study Permit",
  "section": "Bona Fide Student",
  "question": "Which ties to Pakistan can you actually demonstrate with evidence?",
  "help_text": "Strong, documentable ties to Pakistan are the core of your bona fide student case. 'Parents live there' alone is weak. Concrete ties are: property in your name, a family business you are part of, a written return job offer, or financial assets.",
  "input_type": "multi_choice",
  "required": true,
  "options": [
    { "label": "Family ties (parents, spouse, children)", "value": "family_ties" },
    { "label": "Property or financial assets", "value": "property_or_assets" },
    { "label": "Employment or family business", "value": "employment_or_business" },
    { "label": "Career plan in Pakistan", "value": "career_plan_in_pakistan" },
    { "label": "Community obligations", "value": "community_obligations" },
    { "label": "None of the above", "value": "none" }
  ],
  "validation": {
    "pass_if":    { "type": "multi_has_any", "value": ["property_or_assets", "employment_or_business", "career_plan_in_pakistan", "family_ties", "community_obligations"] },
    "trigger_if": { "type": "multi_contains", "value": "none" }
  },
  "show_if": null,
  "mapped_rule": "Bona Fide Student and Intent to Leave Canada",
  "mapped_rule_id": "can_student_bona_fide_student_intent",
  "scoring_key": "bona_fide_intent",
  "score_impact": 8,
  "risk_category": "High",
  "blocker_possible": false,
  "normalized_answer_format": "array_of_strings",
  "error_message": "No demonstrable ties to Pakistan is a near-certain refusal ground. Your application is very high risk without at least one concrete, documentable tie.",
  "source_url": "https://www.canada.ca/en/immigration-refugees-citizenship/services/study-canada/study-permit/eligibility.html",
  "source_resolution_status": "direct_url",
  "last_verified": "2026-05-29"
}
```

**Why this is good:**
- Labels are descriptive; values are clean snake_case
- `"none"` sentinel is present and explicitly handled in `trigger_if`
- `pass_if` lists all positive values — not a negation of `"none"`
- `blocker_possible: false` correctly reflects this is high risk but not a hard stop
- `normalized_answer_format` is `"array_of_strings"` — correct for multi_choice

---

### Example C — Conditional question

```json
{
  "id": "aus_edu_gap_justified",
  "country": "Australia",
  "visa_route": "Student Visa (Subclass 500)",
  "section": "Education",
  "question": "Can you justify your education gap with evidence such as employment letters, exam transcripts, or documented family reasons?",
  "help_text": "A gap over 12 months is not fatal by itself — an unexplained gap is. Provide employment letters, salary slips, freelance or contract proof, or transcripts covering the gap years. 'I was preparing' without documentation is insufficient.",
  "input_type": "yes_no_unknown",
  "required": true,
  "options": [
    { "label": "Yes, I have supporting evidence", "value": "yes" },
    { "label": "No, I cannot justify it", "value": "no" },
    { "label": "Not applicable — I have no gap", "value": "not_applicable" }
  ],
  "validation": {
    "pass_if":    { "type": "in", "value": ["yes", "not_applicable"] },
    "trigger_if": { "type": "eq", "value": "no" }
  },
  "show_if": {
    "question_id": "aus_edu_gap_over_1yr",
    "operator": "eq",
    "value": "yes"
  },
  "mapped_rule": "Educational Consistency and Course Logic (GS-linked)",
  "mapped_rule_id": "aus_student_educational_consistency",
  "scoring_key": "education_consistency",
  "score_impact": 4,
  "risk_category": "High",
  "blocker_possible": false,
  "normalized_answer_format": "string_lowercase",
  "error_message": "An unexplained gap over 12 months is a significant risk at EL3. Gather employment letters, salary slips, or other documentation before lodging.",
  "source_url": "https://immi.homeaffairs.gov.au/visas/getting-a-visa/visa-listing/student-500/genuine-student-requirement",
  "source_resolution_status": "direct_url",
  "last_verified": "2026-05-29"
}
```

**Why this is good:**
- `show_if` is a structured object, not a freeform string
- `not_applicable` is included as an option because some users have no gap
- `pass_if` correctly includes both `"yes"` and `"not_applicable"`
- `"unknown"` is not in the options here — replaced with explicit `"not_applicable"` because the unknown case does not apply semantically

---

## 15. Bad Question Examples

### Bad Example A — Freeform conditional_on string

```json
{
  "question_id": "aus_property_sale_documented",
  "conditional_on": "aus_funds_source = property_sale",
  ...
}
```

**Problems:**
- `conditional_on` is a freeform string — cannot be evaluated programmatically
- Uses old field name `question_id` instead of `id`

**Fix:** Replace with structured `show_if` object:
```json
"show_if": {
  "question_id": "aus_funds_source",
  "operator": "eq",
  "value": "property_sale"
}
```

---

### Bad Example B — Descriptive-text conditional

```json
{
  "question_id": "can_caq_required",
  "conditional_on": "Quebec only",
  ...
}
```

**Problems:**
- `"Quebec only"` has no question ID to evaluate
- This cannot be parsed by a rules engine

**Fix:** Identify which question establishes Quebec status and reference it:
```json
"show_if": {
  "question_id": "can_province_of_study",
  "operator": "eq",
  "value": "quebec"
}
```

---

### Bad Example C — Raw string options

```json
"options": ["yes", "no", "not_sure"]
```

**Problems:**
- No display labels — frontend must guess what to show the user
- Value `"not_sure"` is non-standard — will be renamed to `"unknown"` during cleanup
- Cannot support internationalisation

**Fix:**
```json
"options": [
  { "label": "Yes", "value": "yes" },
  { "label": "No", "value": "no" },
  { "label": "Not sure", "value": "unknown" }
]
```

---

### Bad Example D — Display label used as scoring value

```json
"pass_if": { "type": "eq", "value": "Yes, I have the documents" }
```

**Problems:**
- If the label changes, scoring breaks silently
- This will never match a normalised answer of `"yes"`

**Fix:** Always use the internal `value`, not the `label`:
```json
"pass_if": { "type": "eq", "value": "yes" }
```

---

### Bad Example E — Missing error_message on a blocker question

```json
{
  "id": "uk_has_cas",
  "blocker_possible": true,
  "error_message": "Required."
}
```

**Problems:**
- `"Required."` is a generic placeholder — tells the user nothing
- Does not explain what the blocker is or what the user should do

**Fix:**
```json
"error_message": "A CAS from a licensed UK sponsor is mandatory. No application can proceed without it. An offer letter is not a CAS — request your CAS from the institution after accepting your offer."
```

---

### Bad Example F — Invented mapped_rule_id

```json
{
  "mapped_rule": "Some requirement I vaguely recall",
  "mapped_rule_id": "aus_student_made_up_id"
}
```

**Problems:**
- `"aus_student_made_up_id"` does not exist in `rules.json`
- Linking to a non-existent rule silently corrupts rule-based reporting

**Fix:** If no confident match exists, use:
```json
{
  "mapped_rule": "Some requirement I vaguely recall",
  "mapped_rule_id": null,
  "needs_rule_confirmation": true
}
```

---

### Bad Example G — Fabricated last_verified date

```json
"last_verified": "2024-01-01"
```

**Problem:** This date was not the result of an actual verification — it was typed in as a placeholder. A false `last_verified` date makes stale data appear current.

**Fix:** Use the actual date a human reviewed the question content. If unknown, use `"unknown"` and flag the question for review.

---

## 16. Dataset Cleanup Rules for New Countries

Follow this checklist before committing any new country's `questions.json` to the repository.

### Before writing any questions

- [ ] `rules.json` for the country exists and every rule has a stable `rule_id`
- [ ] `scoring.json` for the country exists and every `category_id` in `scoring_categories` is confirmed
- [ ] The country directory follows the pattern `student/{country_code}/`
- [ ] `student/index.json` has been updated with the new country entry

### Field naming

- [ ] Top-level structure is a bare JSON array (not a wrapped object)
- [ ] Field `id` is used (not `question_id`)
- [ ] Field `question` is used (not `question_text`)
- [ ] Field `help_text` is used (not `user_help_text`)
- [ ] Field `scoring_key` is used (not `score_category`)
- [ ] Field `risk_category` is used (not `risk_level`)
- [ ] Field `show_if` is used (not `conditional_on`)
- [ ] Field `validation` is used (not separate `pass_if` / `trigger_if`)

### Input types

- [ ] No `dropdown` — use `single_choice`
- [ ] No `multi_select` — use `multi_choice`
- [ ] No `yes_no_unsure`, `yes_no_not_sure`, `yes_no_not_yet` — use `yes_no_unknown`
- [ ] No `yes_no_na` — use `single_choice` with explicit `not_applicable` option
- [ ] Currency questions use `currency_amount`, not `number`

### Options

- [ ] Every option is a `{ "label": "...", "value": "..." }` object
- [ ] No raw string arrays in `options`
- [ ] Standard option sets used where applicable (yes/no, yes/no/unknown)
- [ ] No display labels used in `validation`, `show_if`, or scoring references — only `value`s

### Validation

- [ ] Every question has a `validation` object with `pass_if` and `trigger_if`
- [ ] Informational questions use `pass_if: { "type": "any" }` and `trigger_if: { "type": "never" }`
- [ ] No `trigger_if: { "type": "never" }` on `blocker_possible: true` questions

### Conditional logic

- [ ] `show_if` is `null` or a valid structured object — no freeform strings
- [ ] Every `show_if.question_id` references an `id` that exists in the same file
- [ ] No circular `show_if` dependencies

### Scoring and risk

- [ ] Every `scoring_key` value matches a `category_id` in `scoring.json`
- [ ] `blocker_possible` is `true` only for questions with `tier: "hard"` logic
- [ ] `risk_category` uses exactly: `"Critical"`, `"High"`, `"Medium"`, or `"Low"`

### Source references

- [ ] Every question has `source_resolution_status`
- [ ] Questions with `source_url` have `source_resolution_status: "direct_url"`
- [ ] Questions with only `source_ids` have `source_resolution_status: "unresolved"`
- [ ] No question has `source_resolution_status: "missing"` at launch — all sources must be resolved

### Legal confidence

- [ ] Questions where the rule link is uncertain have `needs_rule_confirmation: true` and `mapped_rule_id: null`
- [ ] No question has a `mapped_rule_id` that does not exist in the country's `rules.json`
- [ ] `last_verified` is a real date reflecting an actual human review — not a placeholder

### Final checks

- [ ] `question_count` in `index.json` matches the actual array length
- [ ] All question `id` values are unique within the file
- [ ] JSON parses without error
- [ ] No duplicate `id` values across the entire dataset

---

*End of QUESTION_SCHEMA.md — version 1.0.0*

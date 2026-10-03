---
# NOTE: license left unset intentionally — choose one (e.g. CC-BY-4.0, MIT,
# or CC0) before publishing; not invented here.
pretty_name: Tigrinya Lexical Blind Spot Evaluation — Qwen3-4B-Instruct-2507
language:
  - ti
size_categories:
  - n<1K
tags:
  - tigrinya
  - low-resource-language
  - model-evaluation
  - blind-spot-evaluation
task_categories:
  - translation
  - text-classification
---

# Tigrinya Lexical Blind Spot Evaluation — Qwen3-4B-Instruct-2507

Fatima Fellowship technical challenge submission: **"Blind Spots of Frontier Models."**

## Motivation

Tigrinya is my native language. I noticed that frontier AI systems sometimes seem
surprisingly capable with Tigrinya, but I also know from lived experience that
Tigrinya has regional variation — differences in vocabulary, expressions, and
sometimes very small linguistic distinctions that change meaning entirely
(e.g. **ም ጩ ቑጯቕ** = "to worry about/be concerned about someone" vs.
**ም ጭ ቕጫቕ** = "to argue"). Informal tests with Gemini suggested it sometimes
collapsed such distinctions or mis-translated regional expressions
(e.g. the Mekelle expression for baking/making injera, "injera mggar").

These informal observations were the original motivation, but — importantly —
they were **not assumed to be true**. The goal of this project was to let a
systematic, open-weight model evaluation determine what the actual blind spot
is, rather than forcing the data to confirm the initial idea.

## Research Question (as evolved during testing)

**Initial hypothesis:** Frontier models understand common/written Tigrinya
reasonably well but struggle specifically with regional Tigrinya variation and
locally used expressions.

**Revised hypothesis after baseline testing:** General Tigrinya lexical
knowledge in this model size class is severely limited, and reported model
confidence is not calibrated to correctness. The regional-variation hypothesis
remains **untested/unconfirmed** — see [Limitations](#limitations).

## Model Tested

`Qwen/Qwen3-4B-Instruct-2507` — loaded via Hugging Face Transformers in Google
Colab (T4 GPU). Falls within the fellowship's required 0.6B–6B parameter range.

## Methodology

- Each item was tested with a single run (no ground truth given to the model),
  asking the model to translate/explain the term and self-report a confidence
  score.
- **Basic vocabulary task**: 30 observations across 29 unique Tigrinya lexical
  items. `ገዛ` ("house/home") was intentionally tested twice to examine
  consistency/repeatability of model outputs across identical queries — this
  is a deliberate methodological point, not a data-entry error.
- **Same-meaning judgment task**: 8 word pairs — 4 *contrastive* pairs
  (candidate regional-dialect synonyms) and 4 *control* pairs (words that are
  clearly different in meaning), each judged by the model as same/different
  meaning with an explanation.
- Each item was tested once; temperature/sampling settings were not fixed
  across runs, so determinism is not formally guaranteed (see
  [Limitations](#limitations)).

## Results Summary

### 1. Basic vocabulary accuracy

Excluding the one ambiguous/incomplete item (`ጽቡቕ` — the model's answer
shifted mid-response from "to be/exist" to "uncertain," so it is not cleanly
scoreable as correct/incorrect and is self-annotated "uncertain/incorrect"):

| | Count (n=29) | % |
|---|---|---|
| Correct | 2 | 6.9% |
| Partial | 2 | 6.9% |
| Incorrect | 25 | 86.2% |

Including the ambiguous item as incorrect (n=30): 2 correct (6.7%), 2 partial
(6.7%), 26 incorrect (86.7%) — the result is robust either way.

### 2. Semantic collapse / "attractor" error pattern

Grouping the 26 non-correct/non-partial responses by their actual output text:

| Repeated-answer cluster | Items | Count | % of non-correct answers |
|---|---|---|---|
| "to be / exist" family | ማይ, ገዛ×2, ሰብ, ሰበይቲ, ኣደ, ኣቦ, ዓይኒ, ኣፍ, ክልተ, ሰለስተ, ጽቡቕ, ጸሓይ | 13 | 50.0% |
| "angry / furious" family | ቆልዓ, ኣሓት, እግሪ, ተኽሊ, ጨጉሪ | 5 | 19.2% |
| "to go / move" family | ምግቢ, ሎሚ | 2 | 7.7% |
| Unique, non-repeated wrong answers | እንጀራ, ሓወይ, ሓፍቲ, ኢድ, ርእሲ, መዓልቲ | 6 | 23.1% |

**~77% of all wrong answers collapse into just three repeated semantic
buckets**, rather than being scattered, independent errors. The errors are
heavily concentrated in a small number of repeated semantic outputs rather
than spread evenly across many different meanings — a pattern more consistent
with the model defaulting to a small set of high-frequency "attractor"
outputs for Tigrinya tokens it does not reliably know, than with broadly
varied, independent errors.

A secondary, more specific pattern: some errors show **cross-contamination
within the dataset's own semantic space** rather than a generic default —
e.g. `ሓፍቲ` ("sister") was answered "house/home" (the correct meaning of a
*different* tested word, `ቤት`/`ገዛ`), and `መዓልቲ` ("day") was answered "my
father" (close to the correct domain of `ኣደ`/`ኣቦ`).

### 3. Confidence miscalibration

Confidence was reliably recorded as `5` (maximum) for 27 of 30 items. For
`ኣደ`, `ጽቡቕ`, and `ጸሓይ`, the confidence value was not reliably captured in the
saved output during manual testing (a logging gap — **not** evidence the model
reported a different value), so these 3 items are excluded from the
confidence-calibration analysis below (but retained in the overall accuracy
analysis above).

- **Numerator** (incorrect OR partial, among confidence = 5 responses): 23
  incorrect + 2 partial = **25**
- **Denominator** (all observations with explicitly recorded confidence = 5): **27**
- **25 / 27 = 92.6%** of maximum-confidence responses were not fully correct.
- The remaining 2/27 (7.4%, `ቤት` and `ሓደ`) were fully correct.

Every one of these 27 responses reported confidence = 5 — zero variance — so
confidence carried no discriminative signal about correctness whatsoever in
this sample.

### 4. Same-meaning judgment task (contrastive + control pairs)

Ground truth: all 4 contrastive pairs (`ጨለ`/`እሺ`, `ምህራም`/`ምውቃዕ`, `ህራስ`/`ድቃስ`,
`ምንጋር`/`ምዝራብ`) are **regional variants that share the same meaning**,
confirmed by the annotator as a native speaker. All 4 control pairs are
genuinely **different-meaning** words.

| Pair type | Expected | Model answered | Result |
|---|---|---|---|
| Contrastive (×4) | same | different ("no") | **0/4 correct — model failed every regional-synonym pair** |
| Control (×4) | different | different ("no") | 4/4 correct judgment, but explanations largely unreliable |

- **Important confound**: the model answered "different meaning" (NO) for
  **all 8 of 8 pairs tested, with no exceptions** — both the 4 contrastive
  pairs and the 4 control pairs. Because the contrastive pairs are actually
  same-meaning, this means it got **0/4 contrastive pairs right**, while
  getting 4/4 control pairs right *by judgment alone* (explanations were
  frequently incorrect or uncertain even when the final answer was right).
  But since "NO" was the model's answer in literally every case regardless
  of pair type, **a general default-"different" response bias has not been
  ruled out** — the model may simply answer "no" regardless of input, rather
  than specifically failing to recognize regional synonyms.
- This is a clean, specific result that is **preliminary evidence for**
  Hypothesis B, but it is **not yet a confirmed finding**: distinguishing
  "fails to recognize regional synonyms specifically" from "defaults to
  'different' for everything" requires a positive, non-regional same-meaning
  control pair (correct answer = "YES, same meaning"), which has not been
  tested yet (see [Limitations](#limitations)).

## Hypotheses tracked

| Hypothesis | Status |
|---|---|
| A — General lack of basic Tigrinya lexical knowledge | **Supported** (86%+ error rate on common vocabulary) |
| B — Regional dialect-specific blind spot | **Preliminary evidence, not yet confirmed** — model failed all 4 contrastive (regional-synonym) pairs, but also answered "no" on 100% of pairs tested overall (8/8), so a default-"different" response bias is a live, untested confound |
| C — Confidence miscalibration | **Supported** (92.6% wrong-or-partial despite uniform maximum self-reported confidence) |

## Limitations

- Each item was tested once; temperature was not confirmed fixed across runs,
  so results are not formally guaranteed deterministic. The one repeated item
  (`ገዛ`) produced an identical incorrect answer across both runs, which is
  evidence (though not proof at n=1 pair) that the error pattern is
  stable/structural rather than random sampling noise.
- Confidence values were not reliably captured for 3 of 30 items (`ኣደ`,
  `ጽቡቕ`, `ጸሓይ`) due to a manual logging gap, not a different model value;
  these are excluded from the confidence analysis only.
- No **positive, non-regional** same-meaning control pair (a pair where the
  correct answer is "YES, same meaning" using standard Tigrinya synonyms,
  unrelated to regional dialect) was tested, and this is **not planned** for
  this submission. Without this, it remains fully possible the model simply
  defaults to "different" regardless of input (it answered "no" on all 8/8
  pairs tested, with zero exceptions), rather than specifically failing on
  regional variation. This confound is **not** resolved by the current
  dataset; it is documented here as a known limitation and a candidate for
  future work (see [Proposed path forward](#proposed-path-forward)).
- The regional dialect hypothesis (original motivation) has **preliminary,
  suggestive evidence** from the judgment-task results (0/4 on contrastive
  pairs), in addition to the more general lexical and confidence-calibration
  blind spots found in the basic vocabulary task — but it is not a confirmed
  finding, pending the positive-control test described above as future work.
- Small sample (29–30 lexical items, 8 judgment pairs), single annotator
  (self-reported ground truth), no inter-rater reliability check.
- Early informal piloting (a 25-word pre-cursor run, not included in the
  final 30-item dataset above) also surfaced fabricated-sounding citations in
  some incorrect responses (e.g. "confirmed through standard Tigrinya verb
  dictionaries") — noted here as a qualitative observation for future formal
  study, not quantified in this dataset.

## Proposed path forward

- **Follow-up experiment (not run in this submission)**: test a positive,
  non-regional same-meaning control (standard Tigrinya synonym pairs where
  the correct answer is "YES, same meaning") to determine whether the
  model's "no" answers reflect a regional-dialect-specific blind spot or a
  general default-"different" response bias. This is the single highest-value
  next step for confirming Hypothesis B.
- **Data curation**: collect a larger, dialect-labeled Tigrinya vocabulary and
  same-meaning judgment corpus with balanced YES/NO ground truth, including
  genuine positive non-regional (same-meaning) pairs, to fully remove the
  default-response-bias confound and strengthen the regional-dialect claim
  with more than 4 contrastive pairs.
- **Fine-tuning**: targeted supervised fine-tuning on basic Tigrinya lexical
  pairs to test whether the "attractor" collapse pattern (consistent defaults
  to "to be/exist" and "angry/furious") reflects a fixable vocabulary-coverage
  gap, versus a deeper tokenizer/architecture limitation.
- **Architecture**: investigate how the model's tokenizer fragments Tigrinya
  (Ge'ez/Ethiopic) script, as a possible root cause of the observed lexical
  collapse pattern.

## Files in this repository

- [`basic_vocabulary_results.csv`](basic_vocabulary_results.csv) — 30
  observations (29 unique items) testing isolated-word translation plus
  self-reported confidence.
- [`same_meaning_judgment_pairs.csv`](same_meaning_judgment_pairs.csv) — 8
  pairs (4 contrastive regional-dialect candidates, 4 control) testing
  same/different-meaning judgment.
- `README.md` — this dataset card.

## Open items before publishing

- [x] Fill in the expected same/different relationship for the 4 contrastive
      pairs in `same_meaning_judgment_pairs.csv` — confirmed as "same meaning"
      by the annotator; model answered "different" on all 4 (0/4 correct).
- [x] Positive non-regional same-meaning control experiment — **skipped for
      this submission**; documented as a limitation and proposed follow-up
      instead (see [Limitations](#limitations) and
      [Proposed path forward](#proposed-path-forward)).
- [ ] Choose and add a license.

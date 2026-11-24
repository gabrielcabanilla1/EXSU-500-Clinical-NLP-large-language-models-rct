# Model card – PubMedBERT sentence-role classifier (with context)

> Chosen as the best model on the validation split (`04_model_comparison`); evaluated once on the test split
> (`05_final_test`).

## Model details

- **Model:** `microsoft/BiomedNLP-BiomedBERT-base-uncased-abstract-fulltext` (PubMedBERT, Gu et al., 2021;
  MIT licence), fine-tuned for 5-class sentence classification.
- **Input:** the sentence, paired with a context string: "sentence *i* of *n* . previous: … next: …";
  maximum 256 tokens, truncation of the longest part first.
- **Training:** 2 epochs, AdamW (learning rate 3e-5, weight decay 0.01, linear schedule with 6% warm-up),
  batch size 32, mixed precision, seed 42, one NVIDIA L4 GPU. The best epoch is chosen on the validation
  split.
- **Output:** one probability per class (BACKGROUND, OBJECTIVE, METHODS, RESULTS, CONCLUSIONS); the
  predicted class is the most probable one.
- **Developed by:** EXSU 500 team (McGill University, Fall 2026) as a course project.

## Intended use

- Research and teaching: automatically labelling the rhetorical role of sentences in English abstracts of
  randomised controlled trials, for example to help find the methods or results sentences when screening
  literature.
- Users: researchers and students working on evidence synthesis or clinical NLP.

## Out-of-scope use

- **Any clinical decision.** The model structures text; it does not judge trial quality, validity or
  conclusions.

## Populations and settings on which it should not be used

The model was trained and evaluated only on English abstracts of randomised controlled trials indexed in
PubMed. It should not be used, without new validation, on:

- abstracts of other study designs (observational studies, case reports, systematic reviews, protocols);
- full-text articles, clinical notes, trial registry entries or patient-facing text;
- abstracts in languages other than English, or machine-translated abstracts;
- text from outside biomedicine, or from journals and periods poorly represented in PubMed 20k RCT;
- abstracts whose structure differs strongly from typical RCT abstracts (very short abstracts, or
  sentences that combine several roles).

## Training and evaluation data

PubMed 20k RCT (Dernoncourt & Lee, 2017), official abstract-level splits: 15,000 training, 2,500
validation and 2,500 test abstracts (180,040 / 30,212 / 30,135 sentences). Labels come from the section
headings chosen by the authors of each abstract.

## Metrics

Macro-F1 (primary), accuracy, weighted F1 and, per class, precision, sensitivity, specificity, F1, AUROC
and AUPRC; 95% bootstrap confidence intervals over whole abstracts (2,000 resamples). Calibration:
expected calibration error and reliability diagram.

| Split | Macro-F1 (95% CI) | Accuracy | Macro AUROC | Macro AUPRC | ECE |
|---|---|---|---|---|---|
| Validation | 0.885 (0.879–0.891) | 0.933 | 0.991 | 0.939 | 0.028 |
| Test | 0.886 (0.880–0.893) | 0.930 | 0.990 | 0.935 | — |

F1 by class – validation: BACKGROUND 0.812, OBJECTIVE 0.701, METHODS 0.974, RESULTS 0.970, CONCLUSIONS 0.968;
test: BACKGROUND 0.824, OBJECTIVE 0.703, METHODS 0.968, RESULTS 0.965, CONCLUSIONS 0.971.
Test macro sensitivity 0.883, macro specificity 0.983 (operating point: argmax).
Best epoch: 2 of 2 (0.874 after epoch 1). Training time: 43 minutes on one NVIDIA L4.

Reference models on the validation split (macro-F1): position baseline 0.609; logistic regression with
context 0.852; PubMedBERT without context 0.837. Paired difference against the logistic regression with
context: +0.033 (95% CI 0.028–0.038) on validation and +0.039 (0.034–0.045) on test.

## Known limitations and risks

- **BACKGROUND vs. OBJECTIVE** is the main source of error (61% of validation errors; 35% of OBJECTIVE
  sentences are predicted as BACKGROUND), largely because abstracts use these two headings inconsistently; the
  labels themselves are noisy.
- Slightly overconfident at mid-to-high confidence (ECE 0.028): predicted probabilities should not be read as
  exact error rates.
- PubMedBERT was pretrained on PubMed abstracts that may include test abstracts (possible optimistic bias).
- One training seed; results can vary slightly between runs and GPUs.
- Long inputs are truncated to 256 tokens, which can cut part of the neighbouring sentences.
- Performance may drop on abstracts whose structure differs from typical RCT abstracts.

## Before any real-world use

External validation on abstracts from other sources and years, evaluation on other study designs,
several training seeds, and a review of errors by domain experts would be needed.

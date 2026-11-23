# Rhetorical role classification of sentences in RCT abstracts (PubMed 20k RCT)

EXSU 500 – Fundamentals of Artificial Intelligence in Medicine, McGill University, Fall 2026.
Team project on clinical natural language processing.

> **Course prototype. Not for clinical use.**

## Team

| Member | GitHub | Accountable role | Main code contribution |
|---|---|---|---|
| Gabriel Cabanilla | @gabrielcabanilla1 | Interface lead | PubMedBERT (03c), shared evaluation, model comparison and test notebooks |
| Neda Oghbaei | @nedaoghbaei-afk | Writing lead | Logistic regression (03b) |
| Nyah Woller-Li | @nyahwollerli | Data lead | Position baseline (03a) |

Modelling is shared: each member is accountable for the model family they built.

Detailed contributions (CRediT) and the Use of Generative AI statement are in [`CONTRIBUTIONS.md`](CONTRIBUTIONS.md).

## Problem

Abstracts of randomised controlled trials (RCTs) follow a predictable structure, but many are published
without section headings. The task is to classify **each sentence** of an abstract into one of five
rhetorical roles: **BACKGROUND, OBJECTIVE, METHODS, RESULTS, CONCLUSIONS**. Automatically structuring
abstracts can speed up literature screening and evidence synthesis (for example, finding the results
sentences of many trials quickly).

Task type: multiclass sentence classification (5 classes, imbalanced).

## Data

**PubMed 20k RCT** (Dernoncourt & Lee, 2017), the version with numbers kept (`PubMed_20k_RCT`).
Source: <https://github.com/Franck-Dernoncourt/pubmed-rct>. The abstracts come from MEDLINE/PubMed; the
dataset repository does not state an explicit licence, so the data are **not** redistributed here. They are
downloaded by `02_data_preparation`.

| Split | Abstracts | Sentences |
|---|---:|---:|
| Train | 15,000 | 180,040 |
| Validation (dev) | 2,500 | 30,212 |
| Test | 2,500 | 30,135 |

We use the official splits, which are made **at the abstract level**: no abstract appears in two splits
(checked in `02_data_preparation`). Class shares in the training split: METHODS 33.0%, RESULTS 32.2%,
CONCLUSIONS 15.1%, BACKGROUND 12.1%, OBJECTIVE 7.7%.

## Evaluation protocol

- **Tune on validation, report on test.** Each model chooses its single setting on the validation split
  (position baseline: number of position bins; logistic regression: C; PubMedBERT: best epoch). The test
  split is evaluated **once**, in `05_final_test`, with the frozen models.
- **Primary metric: macro-F1** (every class weighs the same, which matters because OBJECTIVE is only ~8% of
  sentences). Also reported: accuracy, weighted F1 and, per class (one class vs. the other four),
  precision, sensitivity, specificity, F1, AUROC and AUPRC. Operating point: the class with the highest
  predicted probability (argmax).
- **Uncertainty:** 95% bootstrap confidence intervals (2,000 resamples of whole abstracts, seed 42).
  Differences between models use a **paired** bootstrap over the same resampled abstracts.
- **Same conditions for every model:** same prepared data, same splits and one shared evaluation function,
  identical in `03a`, `03b` and `03c`.
- **Calibration and interpretability:** reliability diagrams with expected calibration error, confusion
  matrices with an error analysis, and the most heavily weighted terms of the logistic regression.

## Models

| Family required by the course | Model | Input |
|---|---|---|
| Statistical baseline | **Position baseline**: class frequencies per position bin, learned on the training split | position of the sentence only |
| Classical machine learning | **L2-regularised logistic regression** on TF-IDF (words and word pairs) | sentence; or sentence + previous/next sentence + position |
| Deep learning | **Fine-tuned PubMedBERT** (`microsoft/BiomedNLP-BiomedBERT-base-uncased-abstract-fulltext`, Gu et al., 2021) | sentence; or sentence + position + previous/next sentence |

The two "sentence only / with context" versions form an ablation: they show how much the surrounding
sentences and the position add.

## Repository structure

```
notebooks/
  01_data_exploration.ipynb      dataset description and figures
  02_data_preparation.ipynb      download, checks, shared columns -> prepared data (Google Drive)
  03a_position_baseline.ipynb    statistical baseline
  03b_logistic_regression.ipynb  classical model (two versions)
  03c_pubmedbert.ipynb           deep learning model (both versions in one run, GPU)
  04_model_comparison.ipynb      validation comparison, paired bootstrap, figures, calibration
  05_final_test.ipynb            single evaluation of the frozen models on the test split
  02b_prepare_200k_filtered.ipynb   supplementary: larger training set from PubMed 200k RCT
  03d_pubmedbert_200k.ipynb         supplementary: PubMedBERT trained on that larger set
results/                         tables (.csv) and figures (.png) only; no data, no text of abstracts
MODEL_CARD.md                    model card of the best model
CONTRIBUTIONS.md                 CRediT statement and Use of Generative AI statement
requirements.txt                 Python packages
LICENSE                          MIT (code only)
```

Not committed: the dataset, the prepared CSV files, model weights (PubMedBERT ≈ 420 MB) and prediction
files. They are written to Google Drive (`MyDrive/exsu_project/`).

## How to reproduce

All notebooks run in Google Colab and save their outputs to `MyDrive/exsu_project/` in the Google Drive of
the account that runs them. Run them **in this order, with the same Google account**:

| Step | Notebook | Runtime | Approximate time |
|---|---|---|---|
| 1 | `01_data_exploration` | CPU | 1 min |
| 2 | `02_data_preparation` | CPU | 3 min |
| 3 | `03a_position_baseline` | CPU | 1 min |
| 4 | `03b_logistic_regression` | CPU | 20 min |
| 5 | `03c_pubmedbert` (trains and evaluates both versions in one run) | GPU (L4) | 1–2 h |
| 6 | `04_model_comparison` | CPU | 1 min |
| 7 | `05_final_test` (once, at the very end) | GPU | 5 min |

`03c` skips any version whose trained model is already in Drive, so if Colab disconnects, running it again
continues where it stopped (set `RETRAIN = True` to train again from scratch).
Seeds are fixed (42); PubMedBERT results can still vary slightly between GPU runs.

**Supplementary experiment (not part of the main comparison).** `02b` builds a larger training set from
PubMed 200k RCT after removing every abstract of our validation and test splits (4,862 abstracts), and `03d`
trains PubMedBERT with context on it (one epoch, about 2.15 million sentences). On the same validation split
it reached macro-F1 0.898 (accuracy 0.940), against 0.885 when trained on PubMed 20k RCT. It was evaluated
with its own script, without bootstrap intervals, and never on the test split.

## Results

### Test split (headline results)

Test split (2,500 abstracts / 30,135 sentences), evaluated **once** in `05_final_test` with the models and
settings frozen on the validation split. Macro-F1 with 95% bootstrap CI over whole abstracts.

| Model | Input | Macro-F1 (95% CI) | Accuracy | Macro sensitivity | Macro specificity | Macro AUROC | Macro AUPRC |
|---|---|---|---|---|---|---|---|
| Position baseline (20 bins) | position | 0.603 (0.598–0.610) | 0.686 | 0.615 | 0.915 | 0.915 | 0.623 |
| Logistic regression (C = 4) | sentence | 0.761 (0.754–0.768) | 0.825 | 0.753 | 0.955 | 0.960 | 0.822 |
| Logistic regression (C = 4) | sentence + context | 0.847 (0.840–0.853) | 0.896 | 0.841 | 0.973 | 0.984 | 0.904 |
| PubMedBERT | sentence | 0.827 (0.821–0.834) | 0.881 | 0.823 | 0.970 | 0.980 | 0.893 |
| **PubMedBERT** | **sentence + context** | **0.886 (0.880–0.893)** | **0.930** | **0.883** | **0.983** | **0.990** | **0.935** |

Paired differences in macro-F1 on the test split (all intervals exclude zero):

| Comparison | Difference (95% CI) |
|---|---|
| Logistic regression (sentence) − position baseline | +0.158 (0.149–0.166) |
| Logistic regression: context − sentence only | +0.086 (0.080–0.091) |
| PubMedBERT − logistic regression (sentence only) | +0.066 (0.061–0.072) |
| PubMedBERT: context − sentence only | +0.059 (0.054–0.064) |
| PubMedBERT − logistic regression (both with context) | +0.039 (0.034–0.045) |

F1 by class, best model on test: BACKGROUND 0.824, OBJECTIVE 0.703, METHODS 0.968, RESULTS 0.965,
CONCLUSIONS 0.971. Test results closely match the validation results (largest change: −0.010 macro-F1),
so the model selection did not overfit the validation split.

### Validation split

Validation split (dev, 2,500 abstracts / 30,212 sentences), used to choose every setting.

| Model | Input | Macro-F1 (95% CI) | Accuracy | Macro AUROC | Macro AUPRC | ECE | Training time |
|---|---|---|---|---|---|---|---|
| Position baseline (20 bins) | position | 0.609 (0.603–0.615) | 0.692 | 0.916 | 0.625 | 0.007 | < 1 s (CPU) |
| Logistic regression (C = 4) | sentence | 0.770 (0.763–0.777) | 0.833 | 0.963 | 0.831 | 0.013 | 2.5 min (CPU) |
| Logistic regression (C = 4) | sentence + context | 0.852 (0.845–0.858) | 0.901 | 0.985 | 0.909 | 0.014 | 6.1 min (CPU) |
| PubMedBERT (epoch 2) | sentence | 0.837 (0.831–0.844) | 0.890 | 0.982 | 0.901 | 0.017 | 20 min (L4 GPU) |
| **PubMedBERT (epoch 2)** | **sentence + context** | **0.885 (0.879–0.891)** | **0.933** | **0.991** | **0.939** | 0.028 | 43 min (L4 GPU) |

ECE: expected calibration error (10 confidence bins; lower is better).

Paired differences in macro-F1 on validation (bootstrap over the same resampled abstracts; all intervals exclude zero):

| Comparison | Difference (95% CI) |
|---|---|
| Logistic regression (sentence) − position baseline | +0.161 (0.153–0.170) |
| Logistic regression: context − sentence only | +0.082 (0.076–0.087) |
| PubMedBERT − logistic regression (sentence only) | +0.067 (0.062–0.073) |
| PubMedBERT: context − sentence only | +0.048 (0.043–0.053) |
| PubMedBERT − logistic regression (both with context) | +0.033 (0.028–0.038) |

F1 by class, best model (PubMedBERT + context): BACKGROUND 0.812, OBJECTIVE 0.701, METHODS 0.974,
RESULTS 0.970, CONCLUSIONS 0.968.

Main findings (validation and test agree):
- **Context matters more than model size.** Adding the neighbouring sentences and the position improves
  both models (+8.2 points for the logistic regression, +4.8 for PubMedBERT), and the logistic regression
  with context beats PubMedBERT without it.
- **PubMedBERT with context is the best model**, 3.3 points (validation) and 3.9 points (test) ahead of the
  logistic regression with context, at the cost of a GPU and ~7 times longer training.
- **Most errors are between BACKGROUND and OBJECTIVE** (61% of the best model's 2,035 validation errors
  and 57% of its 2,100 test errors). Abstracts use
  these two headings inconsistently: in the training split, 44.3% of abstracts contain OBJECTIVE but no
  BACKGROUND, 37.0% BACKGROUND but no OBJECTIVE, and 18.5% both.
- All models are well calibrated (ECE ≤ 0.03); PubMedBERT with context is slightly overconfident.

## Interface

Optional (+2%); not built yet.

## Limitations

- The labels come from the section headings chosen by the authors of each abstract, so they are noisy,
  above all BACKGROUND vs. OBJECTIVE.
- Only English abstracts of RCTs indexed in PubMed; performance on other study designs, on full texts or on
  other languages is unknown.
- PubMedBERT was pretrained on PubMed abstracts, which may include abstracts of our test split.
- One random seed per PubMedBERT run; the logistic regression and PubMedBERT encode the context
  differently.
- A sentence classifier structures text; it does not judge the quality or the conclusions of a trial.

## References

1. Dernoncourt F, Lee JY. PubMed 200k RCT: a dataset for sequential sentence classification in medical abstracts. *IJCNLP* 2017.
2. Gu Y, Tinn R, Cheng H, et al. Domain-specific language model pretraining for biomedical natural language processing. *ACM Trans Comput Healthc* 2022;3(1):Art. 2. doi:10.1145/3458754.
3. Jin D, Szolovits P. Hierarchical neural networks for sequential sentence classification in medical scientific abstracts. *EMNLP* 2018.
4. Cohan A, Beltagy I, King D, Dalvi B, Weld DS. Pretrained language models for sequential sentence classification. *EMNLP-IJCNLP* 2019.
5. Pedregosa F, et al. Scikit-learn: machine learning in Python. *JMLR* 2011;12:2825–2830.
6. Wolf T, et al. Transformers: state-of-the-art natural language processing. *EMNLP: System Demonstrations* 2020.
7. Efron B, Tibshirani RJ. *An Introduction to the Bootstrap.* Chapman & Hall, 1993.

## Licence

Code: MIT (see `LICENSE`). The dataset is not included and keeps its original terms.

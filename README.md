# CIC-IDS-2017 Dataset — Cleaned & Split for NIDS Research

| Property | Value |
|----------|-------|
| Records (cleaned) | 2,520,691 |
| Train / Test | 1,924,873 (76.36%) / 595,818 (23.64%) |
| Features | 67 |
| Classes | Binary (benign/attack) + 15 original attack categories |
| Split method | Day-based temporal (Mon–Thu train, Fri test) |
| Data leakage | Zero (verified) |

## Attribution

> Sharafaldin, I., Lashkari, A. H., & Ghorbani, A. A. (2018). Toward generating a new
intrusion detection dataset and intrusion traffic characterization. *ICISSP*.

Dataset source: [Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/ids-2017.html)

## Preparation Pipeline

Run via single Python script, no dependencies beyond
pandas and numpy.

### Summary of steps

- Merge 8 CICFlowMeter CSV shards and extract capture day from filenames
- Standardize column names, handle missing/∞ values and negative artifacts
- Remove zero-duration flows, duplicate and constant columns
- Deduplicate exact duplicate records (keep earliest day occurrence)
- Produce a full cleaned archive and a day-based split (Mon–Thu train, Fri test)

Rationale: CICFlowMeter outputs lack timestamps; day-of-capture is the only reliable
temporal signal. Chronological splitting prevents temporal leakage during evaluation.

## Why Day-Based Temporal Split

Random splits can leak future patterns into training when data are non-stationary.
Using day boundaries (from shard filenames) preserves temporal causality and reflects
realistic deployment scenarios where models are trained on past traffic and tested
on future traffic.

## Class Distribution (post-cleaning)

| Split | Normal | Attack | Normal % | Attack % |
|-------|--------|--------|----------|----------|
| Full  | 2,094,950 | 425,741 | 83.11% | 16.89% |
| Train | 1,719,788 | 205,085 | 89.35% | 10.65% |
| Test  | 375,162 | 220,656 | 62.97% | 37.03% |

Distribution varies by split because Friday contains concentrated DDoS and PortScan
captures in the original collection.

## Files Produced

```
cic_ids2017.csv                Full cleaned dataset (2,520,691 records)
cic_ids2017_train.csv          Train set (1,924,873 records)
cic_ids2017_test.csv           Test set (595,818 records)
cic_ids2017_split_report.csv   Split statistics
cic_ids2017_column_report.csv  Column metadata
```

## Cleaning summary

- Zero-duration flows removed: 2,982
- Duplicate numeric columns removed: 33
- Constant-value columns removed: 1
- Column count: 80 (raw) → 68 (cleaned) → 67 (saved in final CSV)
- Column report: ../cic_ids2017_column_report.csv

## Known Dataset Issues

1. PortScan capture labeling issues documented in Lanvin et al.; these records fall
   primarily in the Friday (test) shards and should be considered when interpreting
   test-set precision.
2. CICFlowMeter export artifacts: duplicate numeric columns (removed) and a small
   number of constant-value columns.
3. Class imbalance: training set is heavily skewed toward benign traffic; apply
   in-training remedies (class weights or oversampling) as needed.

## Downstream Responsibilities (training-time)

- Feature selection: remove memorization-prone fields such as `srcip`, `dstip`, and any absolute timestamps (e.g. `stime`, `ltime`) before training; evaluate feature importance on the training set only.
- Encoding: fit categorical encoders on training data only (one-hot, target, or ordinal encoders as appropriate) and apply the fitted encoders to validation and test sets.
- Normalization / Scaling: fit scalers (StandardScaler/MinMax/etc.) on train and apply to test; do not compute scaling parameters on the full dataset.
- Class imbalance: handle imbalance only on the training set (ADASYN/SMOTE, class weighting, focal loss, etc.); report metrics that are robust to imbalance (precision, recall, F1, and PR-AUC) on the held-out test set.
- Temporal validation: use time-aware cross-validation (temporal folds) for hyperparameter selection to avoid temporal leakage.
- Leakage prevention: fit all preprocessing steps (feature selection, encoders, scalers, oversamplers) inside the training fold; persist transformers with the model if you need to reproduce results.
- Evaluation reporting: publish per-class metrics and confusion matrices in addition to aggregate scores, and document any known labeling caveats affecting the test set.

## References

[1] Arp, D. et al. (2022). Dos and don'ts of machine learning in computer security. *USENIX Sec.*

[2] Corsini, A. et al. (2021). On the evaluation of sequential machine learning for NIDS. *ARES*.

[3] El Mahdaouy, A. et al. (2026). Deep learning for contextualized NetFlow-based NIDS. *arXiv:2602.05594*.

[4] Lanvin, M. et al. (2023). Errors in the CICIDS2017 dataset. *arXiv:2301.04401*.

## License

CIC-IDS-2017 original licensing terms apply; attribute Sharafaldin, Lashkari & Ghorbani (2018) for academic use.

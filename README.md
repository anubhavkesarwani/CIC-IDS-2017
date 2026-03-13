# CIC-IDS-2017 Dataset — Cleaned for NIDS Research

| Property | Value |
|----------|-------|
| Records (cleaned) | 2,520,691 |
| Features | 67 |
| Classes | Binary (benign/attack) + 15 original attack categories |

## Attribution

> Sharafaldin, I., Lashkari, A. H., & Ghorbani, A. A. (2018). Toward generating a new
intrusion detection dataset and intrusion traffic characterization. *ICISSP*.

Dataset source: [Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/ids-2017.html)

## Preparation Pipeline

Run via single Python script, no dependencies beyond
pandas and numpy.

### Summary of steps

- Merge 8 CICFlowMeter CSV shards
- Standardize column names (lowercase, underscores, strip whitespace)
- Identify target label column and drop unlabeled rows
- Fill missing numeric values with 0 and missing categorical values with 'unknown'
- Replace all infinite values (inf, -inf) with 0
- Clamp port numbers to valid range (0–65535)
- Fix negative values in flow_duration and flow rate metrics (bytes/s, packets/s)
- Create binary `attack` column (Benign=0, Attack=1)
- Remove zero-duration flows
- Remove duplicate and constant-value columns
- Deduplicate exact duplicate rows
- Save single cleaned CSV

## Class Distribution (post-cleaning)

| Normal | Attack | Normal % | Attack % |
|--------|--------|----------|----------|
| 2,094,950 | 425,741 | 83.11% | 16.89% |

## Files Produced

```
cic_ids2017.csv                Full cleaned dataset (2,520,691 records)
```

## Cleaning summary

- Missing values filled: 1,358
- Infinite values replaced: 4,376
- Negative flow durations fixed: 115
- Zero-duration flows removed: 2,982
- Duplicate numeric columns removed: 33
- Constant-value columns removed: 1
- Duplicate rows removed: 307,070
- Column count: 79 (raw) → 67 (cleaned)
- Record count: 2,830,743 (raw) → 2,520,691 (cleaned)

## Known Dataset Issues

1. PortScan capture labeling issues documented in Lanvin et al.; these records
   should be considered when interpreting precision.
2. CICFlowMeter export artifacts: duplicate numeric columns (removed) and a small
   number of constant-value columns.
3. Class imbalance: dataset is heavily skewed toward benign traffic; apply
   in-training remedies (class weights or oversampling) as needed.

## Downstream Responsibilities (training-time)

- Feature selection: remove memorization-prone fields such as `srcip`, `dstip`, and any absolute timestamps (e.g. `stime`, `ltime`) before training; evaluate feature importance on the training set only.
- Encoding: fit categorical encoders on training data only (one-hot, target, or ordinal encoders as appropriate) and apply the fitted encoders to validation and test sets.
- Normalization / Scaling: fit scalers (StandardScaler/MinMax/etc.) on train and apply to test; do not compute scaling parameters on the full dataset.
- Class imbalance: handle imbalance only on the training set (ADASYN/SMOTE, class weighting, focal loss, etc.); report metrics that are robust to imbalance (precision, recall, F1, and PR-AUC) on the held-out test set.
- Leakage prevention: fit all preprocessing steps (feature selection, encoders, scalers, oversamplers) inside the training fold; persist transformers with the model if you need to reproduce results.
- Evaluation reporting: publish per-class metrics and confusion matrices in addition to aggregate scores, and document any known labeling caveats affecting the test set.

## References

[1] Arp, D. et al. (2022). Dos and don'ts of machine learning in computer security. *USENIX Sec.*

[2] Corsini, A. et al. (2021). On the evaluation of sequential machine learning for NIDS. *ARES*.

[3] El Mahdaouy, A. et al. (2026). Deep learning for contextualized NetFlow-based NIDS. *arXiv:2602.05594*.

[4] Lanvin, M. et al. (2023). Errors in the CICIDS2017 dataset. *arXiv:2301.04401*.

## License

CIC-IDS-2017 original licensing terms apply; attribute Sharafaldin, Lashkari & Ghorbani (2018) for academic use.

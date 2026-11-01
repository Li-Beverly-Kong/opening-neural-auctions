# Open Results: *Opening the Black Box of Neural Auction Mechanisms*

This repository is the results companion to *Opening the Black Box of Neural Auction Mechanisms*. It releases the archived outputs needed to inspect the paper's empirical claims at three levels:

1. **Feature-level records** - frozen explanations, targets, activation-prediction judgments, and causal metadata.
2. **Auction-level intervention results** - four-dose and six-dose arrays over held-out auctions.
3. **Paper-level summaries** - compact CSV/JSON files corresponding to the reported figures, tables, and headline counts.

This is a **results-only release** organized for inspection and secondary analysis of the evidence reported in the paper.

## Paper and evaluated settings

The release covers exactly the three independently trained settings reported in the paper:

| Setting | Auction environment | Use in the paper |
|---|---|---|
| `2x5c` | 2 bidders, 5 items, continuous public context | Primary SAE, explanation, causal, regret, and diagnostic analyses |
| `2x1c` | 2 bidders, 1 item, continuous public context | Myerson-theory analysis and cross-setting evaluation |
| `2x5d` | 2 bidders, 5 items, discrete context | Cross-setting evaluation |

Every setting has its own CITransNet mechanism and 512-feature TopK SAE. Feature IDs are therefore setting-specific; `F62` in `2x1c` is unrelated to `F62` in `2x5c`.

## Repository layout

```text
opensource_artifacts/
├── README.md
├── raw_results/
│   ├── explanations/{2x5c,2x1c,2x5d}/
│   ├── causal/{2x5c,2x1c,2x5d,2x5c_six_dose}/
│   └── sae_reconstruction/2x5c/
├── derived_results/
│   ├── explanations/
│   ├── causal_audits/
│   ├── myerson_alignment/
│   └── paper_summaries/
└── figures/
```

- `raw_results/` preserves archived experiment outputs with minimal transformation.
- `derived_results/` contains analysis-ready summaries directly tied to paper claims.
- `figures/` contains the five manuscript figures as optional quick-look previews. The machine-readable files, not the rendered PDFs, are the primary released results.

## Suggested reading path

For a quick audit of the paper:

1. Read `derived_results/paper_summaries/` for the reported numerical results.
2. Inspect `derived_results/causal_audits/strong_features_table.csv` for the complete primary 165-feature set.
3. Inspect `raw_results/explanations/<setting>/explanations_512_registry.jsonl` for the feature dictionaries.
4. Use `raw_results/causal/<setting>/causal_effects.npz` for auction-level intervention reanalysis.

## Paper claim to released result map

### A. SAE reconstruction fidelity

**Paper:** Section 4.1, Table 1, Appendix Table 3  
**Setting:** `2x5c`

| File | Contents |
|---|---|
| `raw_results/sae_reconstruction/2x5c/validation_report.json` | Archived representation and downstream reconstruction diagnostics |
| `raw_results/sae_reconstruction/2x5c/activation_count.npy` | Held-out activation counts for all 512 features; supports realized L0 and inactive-feature fraction |
| `raw_results/sae_reconstruction/2x5c/activation_reconstruction_eval.json` | Held-out normalized reconstruction evaluation |
| `derived_results/paper_summaries/sae_reconstruction_summary.csv` | Machine-readable values reported in Tables 1 and 3 |

The released summary covers mean sparsity, standardized reconstruction MSE, inactive-feature fraction, allocation/payment L1 differences, and reconstruction-induced changes in revenue, welfare, allocation entropy, and winner margin.

### B. Frozen explanations and held-out activation prediction

**Paper:** Sections 3.3 and 4.2, Figures 1-2, Appendix Table 4  
**Settings:** all three; the reported pooled control comparison is for primary setting `2x5c`

Each `raw_results/explanations/<setting>/` directory contains:

| File | Contents |
|---|---|
| `explanations.jsonl` | Original generated explanation, selected target, feature ID, and generation metadata |
| `explanations_512_registry.jsonl` | Normalized feature index from ID 0 through 511, with explicit archive status |
| `detection_judgments.jsonl` | Raw held-out binary activation judgments |
| `explanation_validation.parquet` | Canonical per-feature prediction and explanation-validation results |
| `explanation_validation_manifest.json` | Validation schema and run metadata |

Paper-level summaries:

| File | Paper use |
|---|---|
| `derived_results/explanations/cross_split_stability.csv` | Seven target-wise generation/prediction stability results |
| `derived_results/explanations/prediction_count_audit.csv` | Per-feature confusion counts for the seven Figure 1 conditions |
| `derived_results/paper_summaries/prediction_condition_summary.csv` | Pooled precision, recall, and feature-bootstrap confidence intervals |
| `figures/fig_main_phase2_baseline_pr.pdf` | Figure 1 preview |
| `figures/fig_main_phase2_feature_cases.pdf` | Figure 2 preview for F33 and F1 |

### C. Four-dose causal feature validation

**Paper:** Sections 3.4 and 4.3, Figure 3, Appendix Table 5  
**Settings:** all three; Figure 3 and the 165-feature analysis use `2x5c`

Each `raw_results/causal/<setting>/` directory contains:

| File | Contents |
|---|---|
| `causal_effects.npz` | Full 512-feature intervention arrays over four doses and 2,000 held-out auctions |
| `feature_metadata.parquet` | Frozen target, direction, explanation, and feature metadata |
| `bound_claims.parquet` | Frozen claims bound to the intervention run |
| `informative_evidence.parquet` | Feature-level directional tests and dose-response evidence |
| `results.parquet` | Empirical-regret results described in Section D below |

The four-dose NPZ arrays use the following schema:

| Array | Shape | Description |
|---|---:|---|
| `target_values` | `(512, 4, 2000, 7)` | Target value by feature, dose, auction, and stored target |
| `perturbation_norm` | `(512, 4, 2000)` | Hidden-state perturbation magnitude |
| `baseline_feature_activation` | `(512, 2000)` | Dose-1 feature activation |
| `active_mask` | `(512, 2000)` | Baseline-active auction mask |
| `focal_bidder` | `(512, 2000)` | Bidder fixed across doses for bidder-level targets |
| `feature_ids` | `(512,)` | Feature IDs |
| `auction_ids` | `(2000,)` | Held-out auction IDs |
| `doses` | `(4,)` | `[0, 0.5, 1, 2]` |

The final axis of `target_values` is ordered as: `seller_revenue`, `social_welfare`, `bidder_utility`, `payment_to_value`, `allocation_uncertainty`, `winner_margin`, and `incentive_compatibility`. Dose 1 is the SAE reconstruction baseline. An auction is baseline-active for a feature when its maximum token activation is greater than zero; bidder-level targets use the bidder of the baseline maximally activating token, held fixed across doses.

Primary-setting summaries:

| File | Contents |
|---|---|
| `derived_results/causal_audits/strong_features_table.csv` | All 165 strong-response features with target, direction, tests, slopes, monotonicity, prediction metrics, and frozen explanation |
| `derived_results/paper_summaries/strong_feature_representatives.csv` | The 12 Figure 3/Table 5 representatives |
| `figures/fig_b_strong_representatives.pdf` | Figure 3 preview |

### D. Reoptimized empirical-regret audit

**Paper:** Section 4.4 A1, Table 2, Appendix Table 7  
**Settings:** all three

This analysis reruns unilateral-misreport optimization under each edited mechanism. It is not the same as evaluating a forward-pass target.

| File | Contents |
|---|---|
| `raw_results/causal/<setting>/results.parquet` | Feature-level empirical-regret slopes, contrasts, and tests |
| `derived_results/causal_audits/feature_audit_<setting>.csv` | Combined dictionary-level causal and regret audit |
| `derived_results/paper_summaries/regret_summary_by_target.csv` | Primary-setting target-family counts and extreme slopes reported in Table 2 |
| `derived_results/paper_summaries/cross_setting_summary.csv` | Strong-feature and significant regret counts reported in Appendix Table 7 |

### E. Six-dose robustness check

**Paper:** Appendix B.1  
**Setting:** `2x5c`

| File | Contents |
|---|---|
| `raw_results/causal/2x5c_six_dose/causal_effects.npz` | Six-dose arrays with `target_values` shape `(512, 6, 2000, 7)` |
| `raw_results/causal/2x5c_six_dose/feature_metadata.parquet` | Frozen feature metadata |
| `derived_results/causal_audits/feature_audit_2x5c_six_dose.csv` | Dictionary-wide six-dose tests |
| `derived_results/causal_audits/six_dose_dropped_features.csv` | Seven primary strong features not retained at the expanded dose range |
| `derived_results/paper_summaries/six_dose_robustness_summary.csv` | Four-dose/six-dose counts and 158/165 retention result |

The six-dose experiment is a stress test. It is not used to redefine the primary four-dose feature set.

### F. Myerson-theory alignment

**Paper:** Section 4.4 A2, Figure 4, Appendix Table 6  
**Setting:** `2x1c`

| File | Contents |
|---|---|
| `derived_results/myerson_alignment/myerson_ecdf.csv` | Plot-level activations for the three Myerson regions |
| `derived_results/myerson_alignment/myerson_interventions.csv` | Allocation/payment MAE changes and bootstrap intervals |
| `derived_results/paper_summaries/myerson_region_auc.csv` | The five reported evaluation-split region AUCs |
| `figures/fig_myerson_theory_and_intervention.pdf` | Figure 4 preview |

Feature selection for the reported AUCs is performed on the discovery split; the values in `myerson_region_auc.csv` are from the separate evaluation split.

### G. Individual-auction diagnosis

**Paper:** Section 4.4 A3, Figure 5, Appendix Table 8  
**Setting:** `2x5c`

| File | Contents |
|---|---|
| `figures/fig_d_case_diagnostics.pdf` | Allocation and activation-matrix preview for the two cases |

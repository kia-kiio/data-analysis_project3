# Project audit: issues found, fixes applied, verification

## 0. Scope and honest caveats

**Reviewed:** `dry_bean_final.zip` (notebook, tables, figures, README, 5-page PDF, `docs/PROJECT_AUDIT.md`), `dry_bean_final_verified_project.zip` (script-based "repaired" project), and `Dry_Bean_Final_Report.pdf`.

**Not available to me:** the original *requirements PDF* and the "Project B" notebooks were not in the upload, and no screenshots were attached. The requirement checklist in §1 is therefore taken from the task brief, not from the original specification PDF. Please compare it with your PDF once.

**Discrepancy with the brief:** the brief says the final PDF is "around 1 page". The uploaded `Dry_Bean_Final_Report.pdf` was already **5 pages** (the 1-page report belonged to the older Project B, which I cannot see). I rebuilt the report anyway because it lacked required content (below).

**Method:** nothing was taken on trust. The notebook was re-executed from scratch in a clean copy; all 30 regenerated CSV tables and `results_summary.json` matched the shipped ones exactly. Every number in the old PDF prose was then checked against those tables.

## 1. Requirement checklist (from the brief) and status in the final project

| Requirement | Status | Where |
|---|---|---|
| Data quality: missing, duplicates, non-numeric, invalid ranges, class distribution | ✅ | notebook §2–3, report §2–3 |
| Duplicates kept, decision explained and effect measured | ✅ | notebook §6, report §2 |
| EDA + descriptive statistics + 5 plots | ✅ | notebook §4–5, report §4 |
| Split `test_size=0.2, random_state=42, stratify=y` | ✅ | notebook §6 |
| `StandardScaler` fit on Train only | ✅ | notebook §6, leakage audit §14 |
| KNN K = 3/5/9 with Train/Test Acc, Macro Recall, Macro F1, comparison table | ✅ | notebook §7, `tables/knn_comparison_table.csv` |
| No universal "best K" claim | ✅ | CV on Train + standard-error argument |
| 7-class confusion matrix with class names + analysis (best class, lowest recall, top pair, reasons) | ✅ | notebook §8, report §7 |
| K-Means `n_clusters=7, random_state=42, n_init=10`, scaled Train only, predict Test | ✅ | notebook §9 |
| Cluster counts, crosstab, dominant class, **purity** | ⚠️→✅ purity was missing | notebook §9 (new), `tables/kmeans_purity.csv` |
| No raw cluster-ID accuracy | ✅ | documented majority mapping only, clearly labelled |
| Dimensionality/redundancy discussion (Area–Perimeter–EquivDiameter, scaling ≠ decorrelation) | ✅ | notebook §12, report §10 |
| Report 5–7 pages with all sections incl. **conclusion and limitations** | ⚠️→✅ conclusion section was missing | `reports/Dry_Bean_Final_Report.pdf` (6 pages) |
| README (overview, dataset, install, libraries, run, structure, models, results, conclusions) | ⚠️→✅ rewritten | `README.md` |
| Figures, tables, JSON, crosstab, class/cluster plots | ✅ | `figures/`, `tables/` |

## 2. Issues found

Severity: **Critical** = wrong result or violated rule; **Medium** = required content missing or unverifiable claim; **Minor** = quality/polish. No Critical issue was found in the supplied final project.

| # | Sev. | Location | Problem | Why it matters | Fix |
|---|---|---|---|---|---|
| 1 | Medium | notebook §9, report §8, README | **Overall purity of the K-Means clusters was never computed**; only per-cluster dominant-class % existed. | Purity is an explicitly required output. | Added purity cells (per-cluster + overall, Train and Test), majority mapping learned on Train and tested on Test, note that Barbunya is dominant in no cluster. Results: purity 79.9 % (Train) / 80.0 % (Test). New `tables/kmeans_purity.csv`; values added to `results_summary.json`. |
| 2 | Medium | report | **No Conclusion and Limitations section** for the study itself (only "limitations of a packaging line" inside Q9). | Required by the brief; readers cannot judge how far the results generalise. | New report §12: conclusion plus six limitations (single split ±1 %, duplicates/outliers kept, full-data EDA is descriptive only, one imaging setup, K-Means assumptions/purity caveats, no PCA/tuning). |
| 3 | Medium | notebook §6/§13 | **No explicit data-leakage audit.** Rules were partly `assert`ed in scattered places. | Phase 7 asks to verify split, scaler and feature selection. | New notebook §14 with 12 explicit checks (Class not in X; split reproduces `train_test_split(..., stratify=y)`; Train/Test disjoint; scaler saw only Train rows and its mean equals the Train mean but differs from the full-data mean; KNN and K-Means fitted on Train only; K-Means parameters; scaler inside CV pipeline). All pass; a final `assert` fails the run otherwise. |
| 4 | Minor | archive root | Shell brace-expansion bug left a literal folder `{data/raw,notebooks,figures,tables,reports,docs}/` in the zip. | Confusing, unprofessional structure. | Removed. |
| 5 | Minor | old PDF §7 table | Stray `"` character after `Horoz (36.1%)`, and Cluster 6 row dropped `Barbunya (7.7%)`. | Table did not match `kmeans_cluster_table_train.csv`. | Table regenerated from the CSV. |
| 6 | Minor | old PDF §7 bullets | Cluster percentages mix Train and Test without saying so; "≈13 % Dermason in Cluster 4" is the Train value (Test = 14.8 %). | Reader cannot trace the number. | Bullets now state Train/Test explicitly. |
| 7 | Minor | old PDF | All numbers were typed by hand into a Word file. | Risk of drift between report and notebook. | `scripts/build_report.py` builds the report directly from `tables/`; numbers cannot diverge. |
| 8 | Minor | old PDF | Arrows (←) and `(max−min)/min` mirror ambiguously in right-to-left text; two Latin-heavy bullets were hard to read. | Readability. | Replaced with words ("از … به …"); figure order fixed so captions ("right/left") are true for an RTL reader. |
| 9 | Minor | old README | Persian only, no library table, no explicit run instructions, no limitations. | Poor first impression for a reviewer. | Rewritten (English + Persian summary). |
| 10 | Minor | old PDF/report | Only 3 figures; correlation matrix, crosstab heat-map and CV curve were in the notebook but not the report. | Evidence for dimensionality and K discussion. | Added to the report (figures 5–7). |
| 11 | Info | notebook | The shipped notebook outputs were produced by a scripted executor, not a Jupyter kernel (Jupyter is not installable offline in the build sandbox). I used the same approach. | Outputs are real, but not produced by `jupyter nbconvert`. | **Please run *Restart & Run All* once locally**; I expect identical numbers because every random step uses `random_state=42`. |

## 3. Checks that passed without changes (no fix needed)

- Split sizes 10,888 / 2,723; class % within 0.05 pp of the full data.
- `StandardScaler` fitted on Train only; CV scaler inside the pipeline (per fold).
- KNN table columns and values; confusion-matrix pair counts (53 + 51 = 104; Barbunya ↔ Cali = 29).
- K-Means parameters, Train-only fit, `predict` on Test, no cluster-ID accuracy.
- Quoted analytical claims: skewness 2.95; 15 pairs with |r| ≥ 0.95; 4 principal directions for 90 % variance; distance contrast 122.6 → 1.6 → 0.13; Silhouette best at k = 3 (0.406) vs k = 7 (0.313); ARI 0.999 between seeds 42 and 7; unscaled KNN 0.7242 and K-Means ARI 0.378.
- The previous audit's Project A findings are confirmed: `dry_bean_repaired_project` replaces `train_test_split` with a hand-written group-aware splitter (contradicts the specified split), ships no notebook/figures/report, and its own `final_verification.json` says the real dataset pipeline is "NOT VERIFIED". It is superseded and **not** part of the final package.

## 4. Final verification (this delivery)

| Check | Result |
|---|---|
| Notebook executes first to last cell | 35 code cells, 0 errors |
| Re-run in a clean copy reproduces every table and JSON | identical |
| Leakage audit | 12 / 12 checks passed |
| Report length | 6 pages (required 5–7) |
| Report numbers vs `tables/` | all key values found in the PDF text (see final message) |
| Placeholder text in PDF | none |
| README complete | yes |
| Stray folders / caches | none |

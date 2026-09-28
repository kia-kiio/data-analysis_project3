نقش و
نام GitHub

مدیر
Melina Salemi
@s-melina
تحلیلگر
Kiana Sarkari
@kia-kiio
معمار
Marvel Keshtkar
@Marvel200123

# Dry Bean Classification (KNN) and Class-vs-Cluster Comparison (K-Means)

Supervised and unsupervised analysis of the UCI **Dry Bean** dataset (13,611 beans, 16 geometric features, 7 classes).

**Questions answered**
1. Can KNN recognise the bean type from geometric features alone?
2. Do seven K-Means clusters (built without labels) match the seven real classes?

The report is written in Persian (`reports/`); the notebook explanations are Persian and the code, tables and figures are English. A Persian summary is at the end of this file.

---

## 1. Dataset

| Item | Value |
|---|---|
| Source | UCI Machine Learning Repository, *Dry Bean Dataset* (id 602) |
| File | `data/raw/Dry_Bean_Dataset.xlsx` (unmodified) |
| Samples / features | 13,611 rows / 16 numeric features + target `Class` |
| Classes | Barbunya, Bombay, Cali, Dermason, Horoz, Seker, Sira |
| Imbalance | Dermason 26.05 % vs Bombay 3.84 % (ratio 6.79 : 1) |
| Missing / non-numeric / out-of-range | 0 / 0 / 0 |
| Duplicates | 68 repeated rows, all class Horoz, no label conflicts. **Kept**, effect measured (see §5) |

Two raw column names are normalised for consistency: `AspectRation` → `AspectRatio`, `roundness` → `Roundness`.

## 2. Requirements and installation

Python 3.10+ and:

| Library | Tested version |
|---|---|
| numpy | 2.4 |
| pandas | 3.0 |
| scikit-learn | 1.8 |
| matplotlib | ≥ 3.8 |
| seaborn | ≥ 0.13 |
| openpyxl | ≥ 3.1 (reads the `.xlsx`) |
| jupyter | only to open / re-run the notebook |

```bash
pip install -r requirements.txt
```

## 3. How to run

```bash
jupyter notebook notebooks/Dry_Bean_KNN_KMeans.ipynb     # Kernel → Restart & Run All
```

The notebook finds the project root itself (start it from the root or from `notebooks/`), rewrites every file in `figures/` and `tables/`, and ends with self-checks that fail loudly if a methodology rule is broken. All randomness uses `random_state=42`.

To rebuild the PDF report from the generated tables (needs `python-docx` and LibreOffice):

```bash
python scripts/build_report.py      # writes reports/Dry_Bean_Final_Report.docx; convert to PDF with LibreOffice
```

## 4. Project structure

```
dry_bean_final/
├── README.md
├── requirements.txt
├── data/raw/Dry_Bean_Dataset.xlsx
├── data/processed/dry_bean_clean.csv     # analyst output: cleaned data (column names normalised)
├── data/processed/descriptive_statistics.csv
├── notebooks/Analyst_EDA.ipynb           # analyst: data quality + EDA (29 code cells)
├── notebooks/Dry_Bean_KNN_KMeans.ipynb   # complete, executed notebook (35 code cells)
├── scripts/build_report.py               # builds the report from tables/ (no hand-typed numbers)
├── figures/analyst/                      # analyst EDA figures (6 PNG)
├── figures/                              # 13 PNG figures (confusion matrices, crosstab heat-map, class/cluster plots, EDA)
├── tables/                               # 30 CSV tables + results_summary.json
├── reports/Analyst_EDA_Report_FA.pdf     # analyst: 10-page Persian EDA report
├── reports/Dry_Bean_Final_Report.pdf     # 6-page Persian report
├── docs/data_dictionary.csv
├── docs/README_ANALYST_FA.md             # analyst README (Persian)
└── docs/PROJECT_AUDIT.md                 # audit: issues found, fixes, verification
```

## 5. Method summary

| Step | Implementation |
|---|---|
| Features | All 16 numeric columns; `Class` is never an input (asserted) |
| Split | `train_test_split(test_size=0.2, random_state=42, stratify=y)` → Train 10,888 / Test 2,723 |
| Scaling | `StandardScaler` fitted on Train only, applied to Train and Test; inside a `Pipeline` for cross-validation |
| KNN | K = 3, 5, 9 (Euclidean); Train/Test accuracy, macro recall, macro F1, per-class metrics, 7-class confusion matrices; unscaled K = 5 as reference |
| K choice | 5-fold CV on Train only (K = 1…31). No universal "best K" is claimed |
| K-Means | `n_clusters=7, random_state=42, n_init=10`, scaled Train features only, no labels; clusters predicted for Test |
| K-Means evaluation | Cluster sizes, class × cluster crosstab, dominant class and purity per cluster, overall purity, ARI, NMI. **No** raw cluster-ID accuracy |
| Leakage audit | 12 explicit checks in the notebook (§14), all passing |
| Duplicates | Kept. 23 Test rows (0.84 %) have an identical twin in Train; accuracy with / without them: 0.9166 / 0.9167 |

## 6. Results

**KNN (scaled features, Test set)**

| K | Train Acc | Test Acc | Macro Recall | Macro F1 |
|---|---|---|---|---|
| 3 | 0.9499 | 0.9152 | 0.9274 | 0.9285 |
| 5 | 0.9415 | 0.9166 | 0.9271 | 0.9293 |
| 9 | 0.9368 | 0.9163 | 0.9267 | 0.9289 |
| 5, **no scaling** (reference) | 0.8127 | 0.7242 | 0.7229 | 0.7270 |

The three K values differ by 0.0015 accuracy (≈ 4 of 2,723 beans); the standard error of a single accuracy is ≈ 0.005, so the difference is not meaningful.

**Confusion matrix (K = 5):** most correct = Dermason (646); lowest recall = Sira (0.869), then Barbunya (0.875); most confused pair = **Dermason ↔ Sira, 104 errors** (53 + 51), then Barbunya ↔ Cali (29). Bombay is classified perfectly.

**K-Means (k = 7, Test set):** ARI 0.674, NMI 0.722, Silhouette 0.313 (Train), **overall purity 80.0 %**. Bombay, Seker, Dermason and Horoz map cleanly to one cluster (92–100 %); Barbunya and Cali are mixed (clusters 1 and 6); Barbunya is dominant in no cluster. Without scaling ARI falls to 0.378.

## 7. Main conclusions

- Geometric features are enough for ≈ 92 % accuracy / 0.93 macro F1; weakest are the similar-shape pairs Dermason–Sira and Barbunya–Cali.
- Scaling is essential: raw Euclidean distance is dominated by `Area` and `ConvexArea` (≈ 100 % of squared distance).
- K = 3, 5 and 9 are indistinguishable on this split; cross-validation only shows K = 1 is worse.
- K-Means recovers class structure only partially and its cluster IDs are arbitrary labels; it must be judged with crosstab / ARI / NMI / purity, never with accuracy against class labels.
- The 16 features are redundant (4 principal directions carry 90 % of the variance; 15 feature pairs have |r| ≥ 0.95). Scaling fixes units, not redundancy.

## 8. Limitations

Single stratified split (±1 % uncertainty on accuracy); duplicates and outliers deliberately kept; exploratory statistics use the full dataset but never feed a model; features come from one imaging setup, so drift to other cameras/lighting is untested; no PCA, feature selection or hyper-parameter tuning (out of scope); purity depends on the number of clusters and on the majority-vote mapping.

---

## خلاصهٔ فارسی

پروژهٔ تشخیص نوع لوبیا با **KNN** (K = 3، 5 و 9) و مقایسهٔ هفت خوشهٔ **K-Means** با کلاس‌های واقعی روی دیتاست Dry Bean (13٬611 نمونه، 16 ویژگی، 7 کلاس).
اجرا: `pip install -r requirements.txt` و سپس اجرای `notebooks/Dry_Bean_KNN_KMeans.ipynb` با Restart & Run All.
نتیجه: KNN حدود **91.7%** Accuracy و Macro F1 برابر **0.929** (K=5) دارد؛ بیشترین اشتباه بین Dermason و Sira (104 خطا) است. K-Means با ARI = 0.674 و Purity = 80.0% فقط تطابق جزئی دارد و بدون Scale ضعیف‌تر است (ARI = 0.378). گزارش کامل 6 صفحه‌ای در `reports/` و فهرست مشکلات و اصلاحات در `docs/PROJECT_AUDIT.md` است.

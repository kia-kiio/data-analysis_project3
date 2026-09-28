# README — گزارش تحلیلگر داده (Data Analyst)

<div dir="rtl" align="right">

## پروژه طبقه‌بندی انواع لوبیا و مقایسه کلاس‌های واقعی با خوشه‌های K-Means

### 📋 نقش: تحلیلگر داده (Data Analyst)

---

### ۱. معرفی نقش تحلیلگر

وظیفه تحلیلگر داده، آماده‌سازی کامل دیتاست برای مرحله مدل‌سازی توسط **معمار (Architect)** است. این وظایف شامل موارد زیر می‌شود:

- بارگذاری و شناخت داده
- بررسی کیفیت داده (مقادیر گمشده، تکراری، غیرعددی، غیرعادی)
- تحلیل توزیع کلاس‌ها
- آمار توصیفی
- تحلیل تصویری (EDA)
- ماتریس همبستگی
- آماده‌سازی تحویل‌دهی به معمار

---

### ۲. ساختار دیتاست

| مورد | مقدار |
|------|-------|
| نام دیتاست | Dry Bean Dataset |
| منبع | UCI Machine Learning Repository |
| تعداد نمونه‌ها | ۱۳,۶۱۱ |
| تعداد ویژگی‌ها | ۱۶ عددی |
| ستون هدف | Class (۷ نوع لوبیا) |

**هفت کلاس:** BARBUNYA, BOMBAY, CALI, DERMASON, HOROZ, SEKER, SIRA

**ویژگی‌ها:** Area, Perimeter, MajorAxisLength, MinorAxisLength, AspectRatio, Eccentricity, ConvexArea, EquivDiameter, Extent, Solidity, Roundness, Compactness, ShapeFactor1-4

---

### ۳. یافته‌های کیفیت داده

#### مقادیر گمشده
- **هیچ مقدار گمشده‌ای وجود ندارد.** (مطابق مستندات UCI)

#### ردیف‌های تکراری
- **۶۸ ردیف تکراری** شناسایی شد (۰.۵٪ از کل).
- **تصمیم: حذف نشدند.** دانه‌های مختلف لوبیا می‌توانند ویژگی‌های یکسان داشته باشند.

#### مقادیر غیرعددی
- هیچ مقدار غیرعددی در ستون‌های ویژگی وجود ندارد.
- ستون `Class` صحیحاً رشته‌ای (categorical) است.

#### مقادیر غیرعادی (Outliers)
- مقادیر پرت بر اساس IQR شناسایی شدند.
- **تصمیم: حذف نشدند.** مقادیر پرت ممکن است تفاوت‌های طبیعی بین کلاس‌ها باشند (مثلاً اندازه بزرگ BOMBAY).

---

### ۴. توزیع کلاس‌ها

| کلاس | تعداد | درصد |
|------|-------|------|
| DERMASON | ۳,۵۴۶ | ۲۶.۰۵% |
| SIRA | ۲,۶۳۶ | ۱۹.۳۷% |
| SEKER | ۲,۰۲۷ | ۱۴.۸۹% |
| HOROZ | ۱,۹۲۸ | ۱۴.۱۷% |
| CALI | ۱,۶۳۰ | ۱۱.۹۸% |
| BARBUNYA | ۱,۳۲۲ | ۹.۷۱% |
| BOMBAY | ۵۲۲ | ۳.۸۴% |

- **نسبت بزرگ‌ترین به کوچک‌ترین:** ۶.۷۹ برابر
- دیتاست **نامتوازن** است.

---

### ۵. یافته‌های مهم EDA

1. **BOMBAY متمایزترین کلاس است** — اندازه بسیار بزرگ‌تر از سایر انواع.
2. **DERMASON و SIRA بیشترین همپوشانی** را در فضای ویژگی دارند.
3. **Area و Perimeter همبستگی بسیار بالایی** دارند (>0.95).
4. **ویژگی‌های شکل** (Compactness, Eccentricity, Roundness) و **ویژگی‌های اندازه** (Area, Perimeter, ConvexArea) دو گروه مجزای اطلاعاتی تشکیل می‌دهند.
5. **مقیاس ویژگی‌ها بسیار متفاوت است** — Area از ۲۰K تا ۲۵۴K vs Roundness از ۰.۴۹ تا ۰.۹۹.
6. **توزیع Area چوله به راست** است (به خاطر BOMBAY).

---

### ۶. فایل‌های تولید شده

```
data/
├── raw/
│   └── Dry_Bean_Dataset.xlsx          # فایل خام (بدون تغییر)
└── processed/
    ├── dry_bean_clean.csv             # دیتاست با نام ستون‌های اصلاح‌شده
    └── descriptive_statistics.csv     # آمار توصیفی

notebooks/
└── 01_Analyst_Data_Quality_and_EDA.ipynb  # نوت‌بوک تحلیلی

docs/
└── data_dictionary.csv               # فرهنگ داده

figures/
├── class_distribution.png             # نمودار توزیع کلاس‌ها
├── area_histogram.png                 # هیستوگرام Area
├── area_boxplot_by_class.png          # Boxplot Area به تفکیک کلاس
├── area_perimeter_scatter.png         # پراکنش Area vs Perimeter
├── compactness_eccentricity_scatter.png  # پراکنش Compactness vs Eccentricity
└── correlation_matrix.png             # ماتریس همبستگی

reports/
└── Analyst_EDA_Report_FA.pdf          # گزارش PDF فارسی
```

---

### ۷. نحوه اجرای نوت‌بوک

```bash
cd notebooks
jupyter notebook 01_Analyst_Data_Quality_and_EDA.ipynb
```

یا اجرای خودکار:

```bash
jupyter nbconvert --to notebook --execute 01_Analyst_Data_Quality_and_EDA.ipynb
```

**پیش‌نیاز:** کتابخانه‌های زیر باید نصب باشند:
```bash
pip install pandas numpy matplotlib seaborn scikit-learn openpyxl
```

---

### ۸. آنچه معمار (Architect) باید استفاده کند

1. **دیتاست پردازش‌شده:** `data/processed/dry_bean_clean.csv`
2. **تعریف X و y:**
   ```python
   X = df.drop('Class', axis=1)  # 16 ویژگی عددی
   y = df['Class']                # هدف
   ```
3. **تقسیم Train/Test:**
   - `test_size=0.20`
   - `random_state=42`
   - `stratify=y`
4. **Scaling:**
   - `StandardScaler` فقط روی `X_train` آموزش ببیند.
   - سپس روی هر دو `X_train` و `X_test` اعمال شود.
5. **هشدارها:** عدم توازن کلاس‌ها، ویژگی‌های همبسته، مقادیر پرت.

---

### ۹. تصحیحات اعمال‌شده

دو نام ستون در فایل اصلی اشتباه بودند:
- `AspectRation` → `AspectRatio`
- `roundness` → `Roundness`

این تصحیحات در فایل پردازش‌شده اعمال شده‌اند. **فایل خام دست‌نخورده باقی مانده.**

---

*تحلیلگر داده — پروژه طبقه‌بندی انواع لوبیا*

</div>

# Lending Club Loan Default Analysis

## Overview
Analisis risiko default pada 10.000 pinjaman Lending Club melalui 4 dimensi segmentasi menggunakan SQL. Project ini mengombinasikan analisa data dengan perspektif lapangan dari pengalaman sebagai Branch Collection Head di perusahaan multifinance.

## Background
Sebagai ex-Branch Collection Head di WOM Finance, saya menangani proses penagihan dan analisa kredit after sales — mulai dari grading konsumen, proses banding (appeal), hingga pengelolaan akun bermasalah. Project ini bertujuan menguji, menggunakan data publik Lending Club, apakah pola-pola risiko kredit yang saya temui di lapangan juga terlihat di data.

## Dataset
- **Source:** [Lending Club — Kaggle](https://www.kaggle.com/datasets/wordsforthewise/lending-club)
- **Sample:** 10.000 loans, random sample (seed 42) dari 71.454 records
- **Columns:** 9 variabel kunci (grade, purpose, home_ownership, loan_amnt, loan_status, term, int_rate, annual_inc, id)
- **Default definition:** Charged Off + Late (31-120 days)
- **Default rate** = jumlah loan default ÷ seluruh loan dalam segmen (semua status loan ikut di penyebut)

## Business Questions

| BQ | Question | Key Finding |
|---|---|---|
| **BQ1** | Default rate per loan grade (A–G)? | Naik monoton: A (2.53%) → G (41.03%), sekitar **16x** lipat |
| **BQ2** | Purpose mana yang paling sering default? | Debt consolidation (13.28%) tertinggi, di atas credit card (9.70%) |
| **BQ3** | Apakah home ownership mempengaruhi default risk? | RENT (14.81%) lebih tinggi dari OWN (11.35%) dan MORTGAGE (9.72%) |
| **BQ4** | Apakah loan amount besar = default lebih tinggi? | Small ($10k ke bawah) paling rendah (9.63%); Medium (13.57%) dan Large (12.76%) selisihnya kecil dan berbalik tanpa loan Current |

## Key Insights

**1. Grade kredit adalah pembeda terkuat dalam sampel ini.** Default rate naik monoton dari A ke G tanpa anomali, konsisten dengan fungsi grading sebagai dasar risk pricing. Catatan: grade G hanya berisi 39 loan.

**2. Dua temuan yang berbeda dari ekspektasi awal:**
- **BQ2:** Debt consolidation lebih berisiko dari credit card (selisih 3.6 pp, signifikan secara statistik). *Hipotesis dari pengalaman lapangan (belum diuji di data ini):* borrower yang refinancing kemungkinan sudah berada di titik kesulitan finansial lebih dalam ("gali lobang tutup lobang").
- **BQ4:** Medium ($10-20k) sedikit di atas Large (>$20k), tapi selisihnya hanya 0.8 pp dan **belum signifikan secara statistik** (z ≈ 0.9). Pola yang jelas hanya Small yang lebih rendah dari dua segmen lainnya. *Hipotesis dari pengalaman lapangan:* perbedaan kewenangan approval di segmen menengah. Mekanisme ini berasal dari pengalaman di WOM dan belum tentu berlaku di Lending Club.

**3. Home ownership sebagai proxy stabilitas finansial.** RENT lebih tinggi dari OWN (selisih 3.5 pp, signifikan). MORTGAGE sedikit lebih rendah dari OWN, tapi selisih 1.6 pp tergolong lemah (z ≈ 1.7), jadi belum bisa disimpulkan. *Hipotesis untuk diuji:* borrower KPR sudah lolos underwriting bank yang lebih ketat (DSR screening).

## Limitations
- Sampel adalah random sample 10.000 loan dari file sumber 71.454 loan, yang merupakan potongan data, bukan seluruh dataset Lending Club.
- Penyebut default rate mencakup 41.5% loan yang masih berstatus Current. Tanpa loan Current, default rate keseluruhan naik dari 11.92% ke 20.38%; urutan grade tetap monoton, tapi selisih Medium vs Large berbalik (lihat `docs/findings.md`).
- "Default" mencakup Late (31-120 days) yang merupakan kolektibilitas bermasalah, bukan write-off final.
- Beberapa segmen berukuran kecil (misalnya grade G = 39 loan). Perbandingan antar segmen diuji dengan uji dua proporsi sederhana dari angka agregat, bukan model statistik penuh.
- Analisis ini deskriptif (korelasi), bukan sebab-akibat.

## Repo Structure

```
sql-portofolio/
├── README.md              ← anda di sini
├── queries/
│   ├── 01_default_rate_by_grade
│   ├── 02_default_rate_by_purpose
│   ├── 03_home_ownership_impact
│   └── 04_loan_amount_vs_default
├── results/
│   ├── 01_default_rate_by_grade.csv
│   ├── 02_default_rate_by_purpose.csv
│   ├── 03_home_ownership_impact.csv
│   └── 04_loan_amount_vs_default.csv
├── data/
│   └── Lending Club (data).csv
└── docs/
    └── findings.md        ← analisis lengkap + perspektif lapangan
```

## Tools
- SQL (MySQL Workbench)
- Dataset: Lending Club (Kaggle)

## About
**Muhamad Rizieq Zamzani** — Economics graduate (GPA 3.73/4.0), ex-Branch Collection Head at WOM Finance. Transitioning to Data Analyst / Credit Risk Analyst role. Domain expertise in credit risk, collections, grading, and default behavior.

- GitHub: [github.com/rizieqzamzani](https://github.com/rizieqzamzani)
- LinkedIn: [linkedin.com/in/rizieqzamzani](https://linkedin.com/in/rizieqzamzani)

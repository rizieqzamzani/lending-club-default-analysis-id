# LENDING CLUB PORTFOLIO ANALYSIS — FINDINGS

## Ringkasan

Analisis ini menguji risiko default pada 10.000 pinjaman Lending Club (random sample, seed 42, dari 71.454 data) melalui 4 dimensi segmentasi: grade kredit, tujuan pinjaman, status kepemilikan rumah, dan besaran pinjaman. Default didefinisikan sebagai status **Charged Off** atau **Late (31-120 days)**, mencerminkan pinjaman yang sudah masuk fase kolektibilitas bermasalah secara fungsional, bukan hanya write off (WO) final.

**Temuan utama:** Grade kredit adalah pembeda terkuat dan paling konsisten terhadap risiko default (naik monoton A→G, dan tetap monoton walau loan berstatus Current dikeluarkan). Pola RENT vs status lain dan loan kecil vs loan lebih besar juga stabil. Sebaliknya, dua temuan yang awalnya terlihat counter-intuitive (Medium > Large di BQ4 dan MORTGAGE < OWN di BQ3) ternyata lemah secara statistik dan perlu analisis lanjutan.

**Catatan penting soal penyebut:** 41.5% loan di sampel masih berstatus **Current** (belum selesai berjalan). Default rate di dokumen ini dihitung dari seluruh loan, jadi angkanya menggambarkan porsi loan yang sudah bermasalah saat data diambil, bukan rate akhir. Hasil sensitivity check (loan Current dikeluarkan) dicantumkan di tiap BQ.

---

## BQ1: Default Rate by Loan Grade (A–G)

### Hasil
| Grade | Total Loans | Defaulted | Default Rate | Default Rate tanpa Current |
|---|---|---|---|---|
| A | 2,098 | 53 | 2.53% | 4.80% |
| B | 2,775 | 219 | 7.89% | 14.07% |
| C | 2,820 | 372 | 13.19% | 22.33% |
| D | 1,562 | 305 | 19.53% | 31.61% |
| E | 548 | 168 | 30.66% | 42.32% |
| F | 158 | 59 | 37.34% | 46.09% |
| G | 39 | 16 | 41.03% | 50.00% |

### Interpretasi
Default rate naik **progresif dan monoton** dari grade A ke G, dari 2.53% ke 41.03% (sekitar 16x lipat). Pola ini konsisten dengan cara kerja sistem grading Lending Club: semakin tinggi grade risiko yang di-assign di awal, semakin tinggi pula realisasi default-nya.

Saat loan Current dikeluarkan, rate naik di semua grade dan urutannya tetap monoton. Tapi rasio A ke G turun dari sekitar 16x menjadi sekitar 10x, karena porsi loan Current jauh lebih besar di grade A (47%) dibanding grade G (18%). Artinya angka "16x" ikut dipengaruhi komposisi status loan, bukan murni perbedaan risiko.

### Perspektif Lapangan (komparasi pengalaman selama di WOM Finance)
Grading merupakan langkah mitigasi awal untuk meminimalisir risiko dan mempercepat proses analisa konsumen. Dalam pengalaman saya menganalisa konsumen after sales, penerapan grading cenderung menekan angka bad debt, sejalan dengan pola BQ1 di atas.

Salah satu tantangan di lapangan adalah **konsumen banding**: konsumen yang sudah di-reject oleh credit analyst berdasarkan grading, namun kemudian diajukan ulang (banding) oleh tim marketing karena pertimbangan target sales. Konsumen jenis ini sering menjadi problem, baik dari sisi kapasitas bayar aktual maupun karakter kredit.

Grading lebih mudah diterapkan pada konsumen existing (Repeat Order) karena histori pembayaran sudah ada. Untuk konsumen baru, grading tetap bisa diterapkan menggunakan data eksternal seperti SLIK dan Biro Kredit sebagai dasar penilaian riwayat kredit sebelumnya.

### Catatan Data
- Grade G hanya 39 loans (32 tanpa Current). Pola tetap konsisten, tapi ukuran sampel ini perlu disebutkan sebagai limitasi.

---

## BQ2: Default Rate by Loan Purpose

### Hasil
| Purpose Group | Total Loans | Defaulted | Default Rate | Default Rate tanpa Current |
|---|---|---|---|---|
| Debt Consolidation | 5,759 | 765 | 13.28% | 21.33% |
| Others | 1,912 | 201 | 10.51% | 20.47% |
| Credit Card | 2,329 | 226 | 9.70% | 17.63% |

### Interpretasi
Berbeda dari ekspektasi awal (credit card dianggap segmen berisiko tinggi karena mencerminkan borrower yang terjebak siklus utang kartu kredit), hasil justru menunjukkan **debt consolidation** sebagai segmen berisiko tertinggi, sementara **credit card** terendah. Selisih debt consolidation vs credit card (3.58 pp) signifikan secara statistik (z ≈ 4.5), dan urutannya tetap sama tanpa loan Current. Tanpa Current, jarak debt consolidation dengan "others" menyempit (21.33% vs 20.47%).

Hipotesis: borrower yang mengajukan pinjaman untuk konsolidasi utang mungkin sudah berada di titik kesulitan finansial yang lebih dalam. Proses refinancing ini bisa jadi merupakan usaha terakhir sebelum gagal bayar, bukan solusi yang berhasil menyelesaikan masalah utang mereka.

### Perspektif Lapangan (komparasi pengalaman selama di WOM Finance)
*Hipotesis dari pengalaman lapangan, belum diuji di data ini.*

Hal seperti ini banyak terjadi. Umumnya nasabah dalam keadaan mendesak mendapatkan tawaran RO (Repeat Order) dengan kondisi adanya tunggakan di bulan tersebut, sehingga operasional usaha harus mandek karena adanya tagihan usaha yang tidak bisa terealisasi di bulan yang sama. Satu sisi ini strategi marketing yang baik karena melihat peluang kebutuhan konsumen, namun realitanya banyak konsumen yang proses RO dikarenakan kondisi usaha yang sudah memburuk atau menutup hutang di tempat lain (gali lobang tutup lobang) sehingga kemampuan konsumen untuk bayar ke depannya jauh lebih sulit.

### Catatan Data
- Grouping "others" menggabungkan 11 kategori purpose minor (medical, car, house, dll) untuk menghindari sample size terlalu kecil per kategori individual.

---

## BQ3: Home Ownership Impact on Default Risk

### Hasil
| Home Ownership | Total Loans | Defaulted | Default Rate | Default Rate tanpa Current |
|---|---|---|---|---|
| Rent | 3,929 | 582 | 14.81% | 24.73% |
| Own | 1,207 | 137 | 11.35% | 20.21% |
| Mortgage | 4,864 | 473 | 9.72% | 16.78% |

### Interpretasi
Sesuai ekspektasi: borrower dengan status **RENT** memiliki default rate tertinggi (14.81%), sementara **MORTGAGE** terendah (9.72%). Ini sejalan dengan dugaan bahwa kepemilikan aset/collateral berkorelasi dengan stabilitas finansial yang lebih baik. Urutannya tetap sama tanpa loan Current.

Selisih RENT vs OWN (3.46 pp) signifikan secara statistik (z ≈ 3.0). Tapi selisih OWN vs MORTGAGE (1.63 pp) tergolong lemah (z ≈ 1.7), jadi belum bisa disimpulkan bahwa MORTGAGE lebih aman dari OWN. Kalau perbedaan ini terlihat nyata di data yang lebih besar, salah satu hipotesisnya: borrower dengan mortgage sudah melalui proses underwriting bank yang lebih ketat (qualified buyer), sementara status OWN bisa mencakup berbagai skenario (warisan, pembelian tunai tanpa screening kredit ketat, dll) yang tidak seragam kualitas kreditnya.

### Perspektif Lapangan (komparasi pengalaman selama di WOM Finance)
*Hipotesis dari pengalaman lapangan, belum diuji di data ini.*

Status kepemilikan rumah menjadi variabel penting dalam analisa kredit. Nasabah dengan KPR (Mortgage) umumnya melalui proses penilaian yang lebih ketat dari sisi perbankan, termasuk perhitungan **DSR (Debt Service Ratio)**. Selama DSR masih dalam batas wajar meski tanggungan angsuran besar, ini justru jadi indikator bahwa nasabah tersebut layak dibiayai (sudah lolos underwriting ketat).

Di sisi lain, kepemilikan rumah pribadi (OWN) bisa berasal dari berbagai sumber, salah satunya warisan. Ini yang membuat status OWN tidak otomatis mencerminkan histori kredit yang baik, meskipun secara kapasitas finansial bisa jadi nilai plus.

---

## BQ4: Loan Amount vs Default Risk

### Hasil
| Loan Amount Bin | Total Loans | Defaulted | Default Rate | Default Rate tanpa Current |
|---|---|---|---|---|
| Medium ($10k–$20k) | 3,757 | 510 | 13.57% | 22.98% |
| Large (>$20k) | 2,579 | 329 | 12.76% | 23.86% |
| Small (≤$10k) | 3,664 | 353 | 9.63% | 15.67% |

### Interpretasi
Hipotesis awal (pinjaman lebih besar = beban pembayaran lebih berat = risiko default lebih tinggi) hanya sebagian sejalan dengan data. **Small** jelas paling rendah: selisihnya dengan Large (3.12 pp) signifikan secara statistik (z ≈ 3.9).

Sebaliknya, selisih **Medium vs Large** hanya 0.82 pp dan tidak signifikan (z ≈ 0.9). Saat loan Current dikeluarkan, urutannya malah berbalik (Large 23.86%, Medium 22.98%). Jadi kesimpulan yang bisa dipegang: loan kecil lebih aman, sementara Medium dan Large praktis setara dalam sampel ini.

### Perspektif Lapangan (komparasi pengalaman selama di WOM Finance)
*Hipotesis dari pengalaman lapangan di WOM. Mekanisme banding ke Kepala Cabang tidak otomatis berlaku di Lending Club (platform P2P), dan selisih Medium vs Large di data ini belum signifikan.*

Pinjaman skala menengah cenderung lebih rentan karena batas kewenangan approval: jika ditolak oleh credit analyst, keputusan akhir (banding/appeal) di level ini biasanya cukup sampai ke Kepala Cabang saja. Ini membuat proses mitigasi risiko di segmen menengah jadi lebih rentan terhadap kepentingan sales, dibanding pinjaman besar yang biasanya butuh approval di level lebih tinggi dengan scrutiny lebih ketat.

Kalau pola Medium > Large muncul secara konsisten di data yang lebih besar, mekanisme ini bisa jadi salah satu penjelasannya.

---

## Kesimpulan & Rekomendasi Bisnis

1. **Grade kredit tetap jadi indikator paling konsisten** dalam sampel ini, dan pola monotonnya bertahan walau loan Current dikeluarkan. Ini sejalan dengan penggunaan grade sebagai basis utama risk pricing dan keputusan underwriting.
2. **Tujuan pinjaman dan besaran pinjaman memerlukan analisis lebih dalam.** Beberapa selisih antar segmen masih terlalu kecil untuk disimpulkan, dan polanya kemungkinan berinteraksi dengan variabel lain (perlu cross-analysis: purpose × grade, atau loan amount × grade) untuk memahami root cause.
3. **Survey after sales bisa menjadi indikator tambahan dalam sistem grading.** Pengalaman lapangan menunjukkan bahwa kondisi aktual nasabah pasca-pencairan sering berbeda dari profil saat pengajuan. Memasukkan data after sales ke dalam model grading bisa membantu mencegah pola yang sama terulang.
4. **Default rate perlu dibaca dengan konteks status loan.** Dengan 41.5% loan masih Current, analisis lanjutan sebaiknya memakai loan yang sudah selesai berjalan (vintage analysis) agar rate tidak terdilusi.

---

## Data & Metodologi

- **Dataset:** Lending Club (Kaggle), random sample 10.000 dari 71.454 records (seed 42)
- **Kolom kunci:** grade, purpose, home_ownership, loan_amnt, loan_status
- **Definisi default:** Charged Off + Late (31-120 days). Status lain (Fully Paid, Current, In Grace Period, Late 16-30 days, Default) dihitung sebagai non-default. Status "Default" hanya berisi 1 loan.
- **Komposisi status:** Fully Paid 46.0%, Current 41.5%, Charged Off 11.0%, lainnya 1.5%
- **Uji statistik:** two-proportion z-test dari angka agregat; |z| ≥ 1.96 dianggap signifikan pada level 5%
- **Tools:** SQL (MySQL Workbench)
- **Query lengkap:** lihat folder `queries/`

---

## Next Steps

- Cross-analysis grade × purpose dan grade × loan amount. Percobaan awal grade × purpose menghasilkan 74 kombinasi, dengan banyak kombinasi berisi sangat sedikit loan (5 kombinasi hanya 1 loan), jadi perlu ambang minimum jumlah loan sebelum dibandingkan atau diranking.
- Vintage analysis: hitung ulang default rate hanya untuk loan yang sudah selesai berjalan.

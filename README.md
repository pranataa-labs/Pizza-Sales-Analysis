# Business Sales Analysis: Evaluasi Kinerja Penjualan, Pola Transaksi, dan Optimasi Menu

## Headline Hasil Analisis Bisnis

Sepanjang tahun operasional 2015, restoran membukukan total pendapatan kotor (*Gross Revenue*) sebesar **$817.860,05** dari penjualan **49.574 loyang pizza** melalui **21.350 tiket transaksi**. Rata-rata nilai transaksi (*Average Order Value / AOV*) tercatat sebesar **$38,31** dengan konsumsi rata-rata **2,32 loyang per pesanan**. 

Kinerja penjualan didorong secara masif oleh produk ukuran **Large (L)** yang menyumbang **45,9% total omzet ($375.318,70)**, disusul ukuran Medium (30,5%) dan Small (21,8%). Dari sisi waktu, operasional restoran didominasi oleh pola jam sibuk dua gelombang (*Dual-Peak Demand*) yaitu makan siang (*Lunch Rush* 12:00–13:59) dan makan malam (*Dinner Rush* 17:00–19:59) yang secara kumulatif menyumbang **76,2% total volume transaksi harian**. Analisis portofolio menu membuktikan berlakunya prinsip Pareto, di mana **18 dari 32 varian menu (56%) menghasilkan 80% total pendapatan**, dengan *The Thai Chicken Pizza* ($43.434,25) dan *The Barbecue Chicken Pizza* ($42.768,00) sebagai penggerak utama.

![Executive Dashboard](charts/04_sql_executive_dashboard.png)

![Menu Engineering Matrix](charts/16_menu_engineering_matrix.png)

![Peta Panas Aktivitas Transaksi](charts/11_hourly_day_traffic_heatmap.png)

---

## Ringkasan Proyek & Metodologi

Repositori ini memuat analisis eksploratif data mendalam (*Exploratory Data Analysis - EDA*), pemodelan basis data relasional SQLite ternormalisasi, rekayasa fitur (*feature engineering* 150 kolom), serta perancangan strategi bisnis untuk jaringan restoran pizza berdasarkan data transaksi 2015 (`pizza_sales.xlsx`).

Alur kerja analitik mencakup:
1. **Relational Database Staging**: Normalisasi dataset 3NF ke dalam basis data SQLite (`pizza_sales.db`).
2. **Audit Kualitas Data**: Validasi kelengkapan, rekonsiliasi format tanggal campuran (D/M/YYYY dan DD-MM-YYYY), serta integritas matematis finansial (`quantity * unit_price == total_price`).
3. **Advanced Feature Engineering**: Pembentukan tabel analitik `pizza_sales_features_wide` yang memuat **150 kolom analitik** (temporal, geometri ukuran, metrik keranjang belanja, serta matriks indikator biner 65 bahan baku).
4. **SQL Analytics & Window Functions**: Eksekusi query analitik bisnis langsung di atas SQLite (Running totals, MoM Growth, Rank, CTEs).
5. **Menu Engineering Matrix (Kasavana & Smith)**: Klasifikasi portofolio menu menjadi *Stars, Plowhorses, Puzzles, dan Underperformers*.
6. **Supply Chain & Inventory Management**: Pemodelan konsumsi bahan baku dan kalkulasi *Reorder Point (ROP)* serta *Safety Stock*.
7. **Market Basket Analysis**: Ekstraksi aturan asosiasi (*Support, Confidence, Lift*) pada pesanan multi-item untuk perancangan paket combo.
8. **Segmentasi Pesanan Pelanggan**: Pengelompokan pesanan menggunakan algoritma *K-Means Clustering*.
9. **Pengujian Hipotesis Statistik Formal**: Uji ANOVA, Chi-Square Test of Independence, dan Two-Sample T-test pada tingkat signifikansi $\alpha = 0,05$.

---

## Arsitektur Basis Data SQLite (`pizza_sales.db`)

Basis data relasional SQLite `pizza_sales.db` (49,5 MB) memuat 8 tabel terindeks:
- `raw_pizza_sales`: Staging data mentah (48.620 baris x 12 kolom).
- `pizza_sales_clean`: Data transaksi bersih dengan tipe data valid dan tanggal format standar ISO (YYYY-MM-DD).
- `orders`: Tabel pesanan tingkat tiket transaksi (21.350 baris).
- `pizzas`: Katalog produk SKU spesifik per ukuran dan harga (91 SKU).
- `pizza_types`: Katalog konsep varian menu (32 menu).
- `order_details`: Rincian baris transaksi item (48.620 baris).
- `ingredients`: Master data bahan baku (65 jenis bahan).
- `pizza_ingredients_map`: Relasi komposisi bahan baku pada menu (181 relasi).
- `pizza_sales_features_wide`: Tabel analitik kaya fitur (**150 kolom analitik**).

---

## Taksonomi 150 Kolom Analitik (`pizza_sales_features_wide`)

| Domain Fitur | Jumlah Kolom | Atribut Kunci | Deskripsi Analitik |
|:---|:---:|:---|:---|
| **Identitas & Finansial** | 8 | `pizza_id`, `order_id`, `unit_price`, `total_price`, `quantity` | Integritas transaksi dan audit finansial |
| **Waktu & Kalender** | 13 | `order_date`, `order_time`, `order_year`, `order_month`, `order_day_name`, `is_weekend`, `order_hour` | Analisis tren temporal dan musiman |
| **Konteks Operasional** | 6 | `time_period`, `is_lunch_rush`, `is_dinner_rush`, `season`, `is_month_start`, `is_month_end` | Penjadwalan staf dan promosi jam sibuk |
| **Geometri & Efisiensi** | 5 | `size_rank`, `diameter_inch`, `area_sq_inch`, `price_per_sq_inch`, `price_tier` | Evaluasi hukum skala ekonomi ukuran |
| **Agregasi Keranjang** | 8 | `order_total_quantity`, `order_total_revenue`, `order_unique_pizzas`, `basket_tier`, `item_revenue_share` | Profiling perilaku belanja konsumen |
| **Dummy Kategori & Ukuran** | 9 | `cat_classic`, `cat_veggie`, `cat_supreme`, `cat_chicken`, `size_s`, `size_m`, `size_l`, `size_xl`, `size_xxl` | Variabel biner untuk analisis statistik |
| **Matriks Bahan Baku** | 65 | `ing_garlic`, `ing_tomatoes`, `ing_mozzarella_cheese`, `ing_pepperoni`, `ing_chicken`, ... | Pemodelan rantai pasok dan alergen |
| **Dummy Varian Menu** | 32 | `pizza_the_classic_deluxe`, `pizza_the_thai_chicken`, `pizza_the_hawaiian`, ... | Indikator biner 32 menu pizza |
| **Kompleksitas Resep** | 1 | `ingredient_count` | Jumlah bahan per varian menu |

---

## Temuan Kunci & Rekomendasi Strategis

### Temuan Kunci
1. **Dominasi Kategori Classic & Supreme**: Kategori Classic memimpin volume (14.888 loyang, pendapatan $220.053,10), diikuti Supreme ($208.197,00) dan Chicken ($195.919,50). Kategori Chicken mencatat rata-rata harga satuan tertinggi ($17,73).
2. **Efisiensi Geometri Ukuran**: Harga per inci persegi pada ukuran Small adalah $0,157/sq in, sedangkan ukuran Large hanya $0,125/sq in. Ukuran Large memberikan nilai utilitas tertinggi bagi konsumen.
3. **Pola Hari Transaksi**: Jumat ($136.072,90; 3.538 pesanan) dan Sabtu ($123.182,40; 3.158 pesanan) merupakan hari puncak transaksi.
4. **Hasil Pengujian Hipotesis**:
   - ANOVA ($P < 0,001$): Perbedaan rata-rata nilai transaksi antar kategori terbukti signifikan.
   - Chi-Square ($P < 0,001$): Pemilihan ukuran loyang dependen secara signifikan terhadap kategori produk.
   - Two-Sample T-Test ($P = 0,542$): Nilai pengeluaran per tiket antara akhir pekan ($38,40) dan hari kerja ($38,24) tidak berbeda secara signifikan; kenaikan omzet murni didorong oleh frekuensi transaksi.

### 5 Pilar Rekomendasi Aksi
1. **Optimasi Harga**: Menaikkan harga sebesar +$0,50 pada 5 Menu Stars (proyeksi tambahan pendapatan +$18.500/tahun) dan menghapus varian ukuran XXL dari menu Point of Sales (POS).
2. **Penjadwalan Staf Dinamis**: Menambah alokasi koki dan kasir pada jam 11:30–14:00 dan 17:00–20:00, serta menjadwalkan persiapan bahan baku (*pre-prep*) pada jam 10:00–11:00 dan 15:00–16:30.
3. **Pengendalian Persediaan (ROP)**: Menerapkan titik pemesanan ulang bahan baku (Bawang Putih = 185 loyang, Tomat Segar = 178 loyang, Mozzarella = 72 loyang) dan menegosiasikan kontrak pengadaan volume tahunan.
4. **Paket Promo Bundling**: Meluncurkan paket combo *Lunch Express* ($14,99) dan *Family Weekend Feast* (diskon 8%) berbasis *Market Basket Analysis* untuk menaikkan Average Order Value.
5. **Rasionalisasi Menu**: Menghentikan varian *The Brie Carre Pizza* dari menu reguler dan menjadikannya menu musiman (*Limited Time Offer*) untuk menekan biaya bahan baku keju impor berdaya simpan rendah.

---

## Proyeksi Finansial & Roadmap 30-60-90 Hari

- **Skenario Konservatif (+5,5%)**: Tambahan pendapatan tahunan sebesar **+$45.000** (omzet diproyeksikan mencapai **$862.860**).
- **Skenario Moderat (+10,2%)**: Tambahan pendapatan tahunan sebesar **+$84.000** (omzet diproyeksikan mencapai **$901.860**).

| Periode | Fokus Utama | Aktivitas Kunci | Target Metrik |
|:---|:---|:---|:---|
| **Hari 1 - 30** | Operasional & Penataan Menu | Jadwal shift baru jam sibuk; eliminasi menu underperformer dan ukuran XXL; setel parameter ROP inventori. | Waktu tunggu turun 15%; pemborosan bahan turun 3%. |
| **Hari 31 - 60** | Optimasi Harga & Bundling | Kenaikan harga +$0,50 pada 5 Menu Stars; luncurkan 3 paket combo resmi; pelatihan upselling kasir. | AOV naik dari $38,31 ke $40,50. |
| **Hari 61 - 90** | Retensi & Skalabilitas | Program loyalitas pelanggan korporat dan keluarga; review triwulanan margin pengadaan bahan baku. | Pertumbuhan bulanan stabil 8-12%. |

---

## Daftar Berkas & Struktur Direktori

```
C:\Users\User\Desktop\pranataa-lab\datascience\data-analysis\
├── business_sales_analysis.ipynb          # Notebook utama (88 sel terstruktur, tereksekusi 100%)
├── business_sales_analysis_report.md      # Laporan ringkasan manajerial formal
├── Laporan_Analisis_Penjualan_Bisnis_Final.docx # Laporan formal Word (font tabel 8, autofit, indent 1 tab, BAB center)
├── Business_Sales_Analysis_Report.docx    # Salinan laporan formal Word (versi sinkron)
├── Laporan_Analisis_Penjualan_Bisnis.docx    # Laporan lengkap Microsoft Word
├── pizza_sales.db                         # Database SQLite (8 tabel relasional & analitik)
├── pizza_sales_clean.csv                  # Dataset transaksi bersih terstandarisasi ISO
├── pizza_sales_features_wide.csv          # Dataset analitik kaya fitur (150 kolom)
├── 01_import_to_sqlite.py                 # Pipeline ETL dan normalisasi database SQLite
├── build_business_sales_analysis.py       # Script pembangun & eksekutor otomatis notebook
└── charts\                                # Direktori penyimpanan 23 grafik visualisasi PNG
    ├── 01_initial_numeric_boxplots.png
    ├── 02_pizza_geometry_economy_of_scale.png
    ├── 03_pareto_menu_curve.png
    ├── 04_sql_executive_dashboard.png
    ├── ... (23 file visualisasi resolusi tinggi)
```

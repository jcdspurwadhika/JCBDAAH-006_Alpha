# Evaluasi Kinerja Pengiriman, Kepuasan, dan Retensi Pelanggan Olist

**Final Project Kelompok Alpha · Purwadhika**

Analisis data historis e-commerce untuk mengevaluasi hubungan ketepatan waktu pengiriman dengan rating pelanggan, pembelian ulang, dan kontribusi nilai transaksi. Proyek ini menggabungkan data cleaning, exploratory data analysis, pengujian statistik, eksplorasi teks ulasan, dan cohort analysis untuk menyusun prioritas perbaikan bisnis.

## Anggota Kelompok Alpha

1. Muhammad Fatkhan Nur Sholeh
2. Tondi Sudaryo
3. Emmy Jacklyn Pontoan

**[Buka notebook lengkap](<Final_Project_Alpha_.ipynb>)**

## Ringkasan Proyek

Dari **96.446 pesanan delivered yang lolos cleaning**, tingkat keterlambatan mencapai **6,77%**. Pesanan terlambat memiliki rata-rata rating lebih rendah, sementara risiko keterlambatan berbeda menurut wilayah dan periode transaksi. Analisis pelanggan juga menunjukkan bahwa pembelian ulang masih terbatas dalam periode pengamatan.

| Indikator | Hasil |
|---|---:|
| Pesanan dalam master table | 96.446 |
| Pelanggan unik | 93.329 |
| Tingkat keterlambatan | 6,77% |
| Rata-rata keterlambatan pada pesanan terlambat | 10,6 hari |
| Rata-rata rating: tepat waktu / terlambat | 4,29 / 2,27 |
| Total GMV barang pada pesanan delivered | R$13.216.311,19 |
| Repeat buyer | 2.798 pelanggan atau sekitar 3,0% |
| Kontribusi repeat buyer terhadap GMV | 5,5% |

Angka di atas merangkum hasil yang tersimpan dalam notebook. Perbandingan rating menggunakan pesanan yang memiliki skor ulasan. Seluruh nilai uang dinyatakan dalam **real Brasil (BRL/R$)**.

## Pertanyaan Bisnis

1. Apakah rating berbeda antara pesanan tepat waktu dan terlambat? Bagaimana pola kata dan bigram menurut kelompok rating?
2. Wilayah dan seller mana yang menjadi prioritas evaluasi keterlambatan?
3. Bagaimana kontribusi GMV one-time dan repeat buyer, serta hubungan keterlambatan pesanan pertama dengan pembelian ulang?
4. Pada periode mana tingkat keterlambatan meningkat?

Stakeholder utama adalah **Operations & Logistics**, **Customer Experience**, **Seller Management**, dan manajemen strategis.

## Dataset

Sumber: **[Brazilian E-Commerce Public Dataset by Olist — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)**. Analisis menggunakan data historis tahun **2016–2018**.

Notebook memuat delapan CSV berikut:

| File | Informasi utama |
|---|---|
| `olist_orders_dataset.csv` | Status pesanan dan timestamp pengiriman |
| `olist_customers_dataset.csv` | Identitas pelanggan dan wilayah tujuan |
| `olist_order_items_dataset.csv` | Item, harga, seller, dan batas penyerahan |
| `olist_order_reviews_dataset.csv` | Skor bintang dan komentar pelanggan |
| `olist_order_payments_dataset.csv` | Pembayaran |
| `olist_products_dataset.csv` | Atribut produk |
| `olist_sellers_dataset.csv` | Informasi seller |
| `product_category_name_translation.csv` | Terjemahan kategori produk |

Orders, customers, order items, dan reviews menjadi tabel inti analisis. Tabel lain dimuat untuk pemeriksaan awal. Geolocation tercantum pada skema dataset, tetapi tidak dimuat atau dibutuhkan oleh notebook ini; analisis geografis menggunakan `customer_state`.

Unduh CSV dari sumber dataset dan ikuti ketentuan penggunaan pada halaman Olist di Kaggle.

## Persiapan Data

- Memeriksa duplikasi, missing value, tipe data, dan outlier.
- Menyeleksi pesanan `delivered` dengan timestamp penyerahan dan penerimaan yang lengkap.
- Menghapus kronologi ketika penerimaan barang tercatat sebelum penyerahan ke kurir.
- Memilih satu ulasan terbaru per `order_id` berdasarkan timestamp jawaban dan pembuatan ulasan.
- Mengagregasikan item sebelum penggabungan sehingga master table memiliki **satu baris per pesanan**.
- Memvalidasi jumlah baris, keunikan `order_id`, dan total GMV sebelum serta sesudah merge.

| Metrik | Definisi dalam analisis |
|---|---|
| `is_late` | Tanggal penerimaan melewati tanggal estimasi; dibandingkan pada level tanggal |
| `delay_days` | Selisih tanggal penerimaan dan estimasi; nilai positif berarti terlambat |
| `order_gmv` | Jumlah harga barang (`price`) per pesanan, tanpa ongkos kirim |
| Repeat buyer | Pelanggan dengan lebih dari satu pesanan dalam master table, berdasarkan `customer_unique_id` |
| Cohort | Kelompok pelanggan berdasarkan bulan transaksi pertama yang teramati dalam master table |
| Retensi bulan ke-2 | Proporsi pelanggan cohort yang bertransaksi pada bulan kalender setelah bulan transaksi pertama |

## Alur Analisis

| Bagian | Metode dan keluaran |
|---|---|
| Sesi 1 — Kepuasan | Rata-rata rating, komposisi bintang, dan Welch's independent t-test |
| Sesi 1 Lanjutan — Teks ulasan | Cakupan komentar, bias partisipasi, word count, bigram, serta distribusi dan komposisi sentimen berbasis rating |
| Sesi 2 — Logistik | Kategori keterlambatan berdasarkan timestamp, perbandingan wilayah, evaluasi seller, dan uji Chi-Square |
| Sesi 3 — Retensi dan GMV | Cohort heatmap, segmentasi one-time/repeat buyer, kontribusi GMV, dan uji hubungan keterlambatan pesanan pertama dengan repeat order |
| Sesi 4 — Tren | Volume transaksi dan tingkat keterlambatan per bulan |
| Rekomendasi | Prioritas tindakan per stakeholder, target SMART, dan skenario cost-benefit |

Pada analisis teks, **rating 1–2 = Negatif, 3 = Netral, dan 4–5 = Positif**. Label ini merupakan proxy skor bintang, bukan hasil model klasifikasi sentimen kalimat. Bigram dibentuk setelah preprocessing dan penghapusan stopword.

## Temuan Utama

**1. Kelompok terlambat memiliki rating lebih rendah.** Rata-rata rating adalah **2,27**, dibandingkan **4,29** pada kelompok tepat waktu. Proporsi bintang 1 masing-masing sebesar **53,78%** dan **6,61%**. Welch's t-test menunjukkan perbedaan signifikan pada ambang 5%.

**2. Keterlambatan terkonsentrasi pada wilayah dan periode tertentu.** Late rate mencapai sekitar **21,4% di Alagoas**, **17,4% di Maranhão**, dan **15,2% di Sergipe**. Pada analisis bulanan, Maret 2018 mencatat **18,96%**, Februari 2018 **14,13%**, dan November 2017 **12,40%**.

**3. One-time buyer menyumbang sebagian besar GMV.** Kelompok ini berkontribusi **94,5%** terhadap GMV. Rata-rata GMV kumulatif per repeat buyer sebesar **R$259,97**, dibandingkan **R$137,95** per one-time buyer selama periode pengamatan.

**4. Pengalaman pesanan pertama berkaitan dengan pembelian ulang.** Repeat rate sebesar **3,04%** pada kelompok pesanan pertama tepat waktu dan **2,49%** pada kelompok pertama terlambat. Uji Chi-Square menghasilkan **p-value 0,0149**.

**5. Komentar tidak mewakili seluruh kelompok rating secara merata.** Dari **95.800 ulasan**, sebanyak **40.564 (42,3%)** menyertakan teks. Persentase penyertaan komentar mencapai **76,0% pada rating 1–2**, dibandingkan **36,7% pada rating 4–5**. Di antara ulasan berteks, proporsi rating 1–2 mencapai **72,5% pada kelompok terlambat** dan **17,8% pada kelompok tepat waktu**.

## Arah Rekomendasi

| Stakeholder | Fokus tindakan yang diusulkan |
|---|---|
| Operations & Logistics | Evaluasi proses dan SLA pada wilayah prioritas; perencanaan kapasitas untuk periode berisiko |
| Customer Experience | Pilot notifikasi keterlambatan dan kompensasi tertarget, disertai evaluasi pembelian ulang |
| Seller Management | Pembinaan seller dengan tingkat keterlambatan penyerahan tinggi |
| Manajemen | Evaluasi alokasi anggaran akuisisi dan retensi berdasarkan hasil pilot |

Target dalam notebook adalah menurunkan late rate dari **6,77% menjadi ≤5,75%** dan meningkatkan repeat rate dari sekitar **3,00% menjadi ≥3,15%** dalam **12 bulan**. Target tersebut adalah perubahan sekitar **15% relatif** dan **5% relatif**, bukan penurunan atau kenaikan sebesar 15 dan 5 poin persentase.

Target dan estimasi cost-benefit merupakan **usulan berbasis skenario**, bukan hasil intervensi yang telah terealisasi. Asumsi biaya, penebusan voucher, dan transaksi tambahan perlu divalidasi melalui pilot.

## Teknologi

**Python · pandas · NumPy · Matplotlib · Seaborn · SciPy · Jupyter Notebook · VS Code**

Modul bawaan `os`, `re`, dan `collections.Counter` digunakan untuk akses file serta pengolahan teks sederhana.

## Menjalankan di VS Code

### 1. Siapkan lingkungan Python

Pasang Python serta ekstensi **Python** dan **Jupyter** dari Microsoft di VS Code. Buka folder proyek, lalu pasang dependency pada lingkungan Python yang akan digunakan:

```bash
python -m pip install pandas numpy matplotlib seaborn scipy ipykernel
```

### 2. Siapkan dataset

Simpan delapan CSV yang tercantum di atas dalam satu folder. Pastikan nama file sesuai dengan nama yang dipanggil kode; misalnya `olist_order_items_dataset.csv`, tanpa tambahan `(1)`.

Pada cell loading, ubah `DATA_DIR` ke lokasi dataset milikmu. Contoh Windows:

```python
import os

DATA_DIR = r"D:\OlistProject\data"

def load(name):
    return pd.read_csv(os.path.join(DATA_DIR, name))
```

Notebook menggunakan path lokal yang harus disesuaikan pada komputer lain. Jalankan cell import terlebih dahulu agar `pd` tersedia. Jika folder `data` berada pada direktori kerja notebook, kamu juga dapat menggunakan `DATA_DIR = "./data"`.

### 3. Jalankan notebook

1. Buka `Final_Project_Alpha(3).ipynb`.
2. Pilih **Select Kernel** dan gunakan lingkungan Python tempat dependency dipasang.
3. Jalankan seluruh cell secara berurutan dari atas melalui **Run All**.
4. Setelah mengubah kode, restart kernel dan jalankan kembali seluruh cell sebelum menyimpan hasil akhir.

Panduan editor: [Jupyter Notebooks in VS Code](https://code.visualstudio.com/docs/datascience/jupyter-notebooks).

Daftar isi di dalam notebook masih menggunakan tautan `#scrollTo=...` dari Colab. Untuk navigasi di VS Code, gunakan panel **Outline**. Tautan tersebut tidak memengaruhi perhitungan.

## Batas Interpretasi

- Data bersifat observasional: hubungan antarvariabel dan signifikansi statistik belum memastikan sebab-akibat.
- Populasi hanya mencakup pesanan delivered yang lolos cleaning. Pesanan gagal terkirim atau dibatalkan tidak termasuk analisis utama.
- Kategori seller/kurir berasal dari aturan timestamp, bukan verifikasi langsung tanggung jawab operasional; pesanan multi-seller juga membatasi ketepatan atribusi.
- Tidak membeli ulang dalam periode dataset tidak berarti churn permanen. Lama pengamatan pelanggan dan cohort berbeda-beda.
- Analisis teks hanya mencakup ulasan berkomentar. Stopword removal dapat menghilangkan konteks; frekuensi kata dan bigram belum memastikan tema atau penyebab keluhan.
- GMV bukan laba, dan GMV kumulatif per pelanggan bukan lifetime value penuh. Data biaya serta margin belum cukup untuk menyimpulkan ROI aktual.

## Berkas Proyek

| Berkas | Isi |
|---|---|
| [Final_Project_Alpha(3).ipynb](<Final_Project_Alpha(3).ipynb>) | Kode analisis, visualisasi, interpretasi, dan rekomendasi |
| [README.md](README.md) | Ringkasan proyek dan panduan penggunaan |

Letakkan README dan notebook pada folder utama repository. Jika nama notebook diubah, sesuaikan tautan notebook pada README.

## Kredit

Disusun oleh **Kelompok Alpha** sebagai final project Purwadhika. Dataset disediakan oleh **Olist** melalui [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce). Proyek ini merupakan studi untuk pembelajaran dan portofolio, bukan laporan resmi Olist.

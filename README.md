# Prediksi Inflasi Bulanan 5 Kota dengan LSTM: Skripsi dan Uji Ulang

Skripsi S.Kom. saya (Teknik Informatika, Universitas Muhammadiyah Magelang, 2026) memprediksi inflasi bulanan Medan, DKI Jakarta, Bandung, Surabaya, dan Makassar dengan LSTM, ditambah UMR sebagai variabel pendukung dan tuning hiperparameter dengan Optuna.

Setelah lulus, saya menjalankan ulang seluruh eksperimen dengan evaluasi yang lebih ketat. Hasilnya tidak sebagus yang saya laporkan di skripsi. Repo ini berisi kode uji ulang, audit penyebabnya, dan arsip notebook skripsi.

**Halaman studi kasus (interaktif):** https://margoanan.github.io/inflasi-lstm-5-kota/

## Temuan utama

- Di skripsi, MSE Kota Medan turun 55% dari baseline ke model final.
- Saat diuji ulang tanpa memakai data uji untuk memilih model, model final **tidak lebih baik dari menebak rata-rata 12 bulan terakhir** di 3 dari 5 kota, dan hanya unggul kurang dari 3% di dua kota lainnya.
- Penyebab utamanya: data uji ikut dipakai untuk memilih model, tidak ada pembanding sederhana, dan inflasi bulanan memang hampir tidak bisa ditebak dari masa lalunya (autokorelasi lag 1 hanya 0,08–0,30).

## Hasil uji ulang (MSE data uji, Okt 2020 – Des 2025)

| Kota | Rata-rata 12 bulan | LSTM baseline | + UMR | + UMR + Optuna | Skripsi (Optuna) |
|---|---|---|---|---|---|
| Medan | **0,3020** | 0,5417 | 0,5193 | 0,3380 | 0,2712 |
| Jakarta | 0,2019 | 0,1941 | **0,1831** | 0,2010 | 0,1285 |
| Bandung | 0,2727 | 0,5616 | **0,2617** | 0,3544 | 0,2449 |
| Surabaya | **0,1525** | 0,4083 | 0,1862 | 0,1605 | 0,1377 |
| Makassar | 0,3088 | 0,3763 | **0,2838** | 0,3016 | 0,2613 |

Kolom "Skripsi" adalah angka yang dilaporkan di skripsi (Tabel 4.5, 4.7, 4.9). Angka itu dipilih dengan melihat data uji, jadi tidak ikut dibandingkan. UMR pada tabel ini adalah versi level. Hasil lengkap, termasuk versi pertumbuhan UMR, ada di `outputs/summary_test_mse.csv`.

## Isi repo

```
data/            inflasi_umr_5kota_2000_2025.csv  (BPS, 5 kota x 312 bulan)
notebooks/
  01_eksperimen_lstm.ipynb   Uji ulang: pembanding sederhana, Skenario 1–3, untuk 5 kota sekaligus
  02_audit.ipynb             Audit penyebab (tidak butuh TensorFlow)
outputs/         Hasil 01_eksperimen_lstm.ipynb: metrik, prediksi, parameter terbaik, grafik
docs/            Halaman studi kasus (GitHub Pages)
arsip_skripsi/   Notebook asli skripsi, disimpan apa adanya sebagai bahan audit
```

## Cara evaluasi di uji ulang

- Data dibagi berurutan: **80% latih** (Jan 2000 – Sep 2020) dan **20% uji** (Okt 2020 – Des 2025), sama dengan skripsi.
- Scaler selalu di-fit hanya pada data latih.
- Skenario 1 dan 2 memakai konfigurasi dan jumlah epoch dari artikel acuan, tanpa early stopping.
- Skenario 3: Optuna memilih hiperparameter dengan **validasi silang walk-forward 3 lipatan di dalam data latih** (Okt 2014 – Sep 2020). Data uji hanya dipakai sekali di akhir.
- UMR diuji dalam dua versi: level (apa adanya) dan pertumbuhan tahunan (%).
- Pembanding sederhana: bulan lalu, rata-rata historis, rata-rata 12 bulan terakhir, dan nilai 12 bulan lalu.

## Menjalankan

**Google Colab:** buka `notebooks/01_eksperimen_lstm.ipynb`, unggah `data/inflasi_umr_5kota_2000_2025.csv`, lalu Run all. Skenario 3 butuh sekitar 2–4 jam dengan GPU. Aktifkan sel Google Drive agar studi Optuna bisa dilanjutkan bila sesi terputus.

**Lokal:**

```bash
pip install -r requirements.txt
jupyter notebook notebooks/
```

Notebook dijalankan dengan seed 42, tetapi hasil LSTM bisa sedikit berbeda antar-perangkat dan antar-versi TensorFlow.

## Tentang arsip skripsi

Notebook di `arsip_skripsi/` adalah versi yang saya pakai selama pengerjaan skripsi. Setiap kota dijalankan terpisah dengan mengganti variabel `city`. Masalah yang ditemukan saat audit:

- data uji dipakai sebagai `validation_data`, untuk memilih hiperparameter (Skenario 2), dan sebagai nilai objektif Optuna (Skenario 3);
- scaler di Skenario 2 di-fit pada seluruh data, termasuk periode uji;
- beberapa pengaturan di notebook tidak sama dengan naskah skripsi (misalnya batch size dan pilihan GRU di Skenario 3).

## Penulis

Afnan Ahsana Syadida, S.Kom. · [linkedin.com/in/afnanahsana](https://www.linkedin.com/in/afnanahsana)

Data: Badan Pusat Statistik (inflasi bulanan dan UMR), Jan 2000 – Des 2025.

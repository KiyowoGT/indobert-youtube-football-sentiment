# Analisis Sentimen Komentar YouTube Laga Portugal vs Kroasia Menggunakan *Transfer Learning* IndoBERT

Repositori ini memuat dataset, kode eksperimen, dan skrip penyusun naskah untuk klasifikasi sentimen komentar YouTube berbahasa Indonesia pada laga Portugal melawan Kroasia, memakai arsitektur *Transfer Learning* IndoBERT, beserta pemodelan regresi dampak insiden VAR (*Video Assistant Referee*).

Seluruh angka di naskah dihasilkan oleh notebook pada repositori ini, bukan dihitung manual.

---

## 📁 Struktur Repositori

```text
.
├── data/
│   ├── raw/
│   │   └── youtube_comments.csv                  # tarikan mentah YouTube Data API v3 (5.460 baris)
│   └── processed/
│       ├── dataset_youtube_labeled_all.csv       # 3.316 komentar berlabel (korpus penuh)
│       ├── dataset_dibuang.csv                   # komentar tereliminasi + alasan
│       ├── dataset_komentar_indonesia_bersih.csv # keluaran tahap saring bahasa
│       ├── train_dataset_80.csv                  # 2.652 baris latih
│       ├── test_dataset_20.csv                   # 664 baris uji
│       ├── validasi_label_manual.csv             # 150 baris, kolom label_manual kosong
│       └── validasi_label_manual_terisi.csv      # 150 baris, sudah terisi (dasar hitung Kappa)
├── validation/
│   ├── 01_Preprocessing_Penyortiran_Split80_20.ipynb
│   ├── 02_Validasi_Metode_4Paper_ke_Dataset_YouTube.ipynb
│   └── 03_Validasi_Sentimen_IndoBERT_Revisi.ipynb
├── replication/
│   ├── Replikasi_Lolos/                          # 4 paper yang berhasil direplikasi
│   └── Replikasi_Tidak_Lolos/                    # 9 paper yang gagal direplikasi
└── README.md
```

Repositori ini sengaja hanya memuat dataset, kode validasi, dan kode replikasi. Berkas naskah `.docx`, skrip penyusun naskah, dan kerangka templat dikelola terpisah.

* **`validation/02_Validasi_Metode_4Paper_ke_Dataset_YouTube.ipynb`** — notebook utama. Melatih IndoBERT, menjalankan empat *baseline* klasik, dekomposisi matriks konfusi, ROC/PR, Stratified 5-Fold CV, uji Wilcoxon dan uji-t berpasangan, pengukuran latensi, serta regresi logistik insiden VAR. Memuat aturan kata kunci terpusat sehingga `penaldo` tidak terhitung sebagai penalti dan `variasi` tidak terhitung sebagai VAR. Menghasilkan angka yang dipakai naskah.
* **`validation/03_Validasi_Sentimen_IndoBERT_Revisi.ipynb`** — notebook revisi yang lebih baru. Menambah audit komentar kembar latih→uji, audit label sarkasme berbasis penanda tawa, tiga *seed* (42/43/44), pencatatan versi pustaka, dan pembandingan aturan kata kunci VAR lama lawan baru. Menyimpan seluruh tabel keluaran ke `hasil_validasi/` beserta `ringkasan_angka_paper.json`.
* **`data/processed/validasi_label_manual_terisi.csv`** — 150 label manual beserta label otomatis, dasar hitung kesepakatan 46,00% dan Cohen's Kappa 0,19. Versi kosongnya (`validasi_label_manual.csv`) disertakan agar alur anotasi bisa diulang.
* **`validation/01_Preprocessing_Penyortiran_Split80_20.ipynb`** — penyaringan bahasa dua sisi dan split terstratifikasi 80:20 dengan `random_state=42`. Membaca `data/raw/youtube_comments.csv` dan menuliskan seluruh korpus ke `data/processed/`.
* **`replication/`** — notebook replikasi keempat paper acuan pada dataset aslinya, dipakai untuk menetapkan metode mana yang layak dipindahkan ke korpus YouTube. Empat lolos, sembilan tidak.

---

## 🚀 Cara Menjalankan

### Opsi 1: Lokal (disarankan)

```bash
git clone https://github.com/KiyowoGT/indobert-youtube-football-sentiment
cd indobert-youtube-football-sentiment
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install "transformers>=4.30.0" "torch>=2.0.0" "scikit-learn>=1.2.0" \
            "statsmodels>=0.14.0" seaborn matplotlib scipy psutil pandas numpy jupyterlab
jupyter lab validation/
```

Jalankan notebook `01` lebih dulu, lalu `02`. Keduanya memakai path relatif terhadap direktori kerja, jadi jalankan Jupyter dari akar repositori atau sesuaikan variabel path di sel pemuatan data.

### Opsi 2: Google Colab

1. Buat folder `dataset` di direktori utama Google Drive.
2. Unggah `data/processed/train_dataset_80.csv` dan `data/processed/test_dataset_20.csv` ke folder tersebut.
3. Jalankan notebook; kode akan *mount* Drive dan membaca dari `/content/drive/MyDrive/dataset/`.

---

## ⚠️ Dua notebook, dua set angka

`validation/02` dan `validation/03` **tidak menghasilkan angka yang sama**. Notebook 02 adalah dasar seluruh angka pada naskah; notebook 03 adalah revisi yang lebih baru dan belum dipakai di naskah.

| Metrik (data uji) | Naskah & notebook 02 | Notebook 03 |
|---|---|---|
| Akurasi | 74,85% | 78,77% |
| Macro-F1 | 72,56% | 76,70% |
| ROC-AUC | 89,24% | 89,90% |
| Latensi | 5,434 ms | 5,03 ms |
| *Throughput* | 184,0 RPS | 198,8 RPS |
| RAM | 2.041,1 MB | 1.817,6 MB |
| *Seed* | 42, 123, 2024 | 42, 43, 44 |
| Akurasi antar-*seed* | 76,66 ± 2,07 | 78,97 ± 0,40 |
| Uji Wilcoxon / uji-t | ada | **tidak ada** |

Penyebab utama: notebook 03 memperbaiki aturan kata kunci kategori VAR dan aspek entitas, sehingga Tabel 1, Tabel 2, dan koefisien OLS ikut bergeser. Notebook 03 juga mengganti *seed* dan menghapus uji signifikansi.

**Konsekuensinya:** angka di naskah belum tentu cocok dengan keluaran notebook 03. Kalau dosen menjalankan notebook 03 lalu membandingkan dengan naskah, selisihnya akan terlihat. Putuskan satu: perbarui naskah memakai angka 03, atau tandai 03 sebagai eksplorasi dan tetap pakai 02.

---

## ⚙️ Lingkungan Komputasi

| Komponen | Nilai |
|---|---|
| Python | 3.10 (runtime Google Colab) |
| PyTorch | ≥ 2.0.0 |
| Transformers | ≥ 4.30.0 |
| scikit-learn | ≥ 1.2.0 |
| statsmodels | ≥ 0.14.0 |
| Perangkat keras | GPU NVIDIA CUDA (Tesla T4, Colab) |
| Model dasar | `indobenchmark/indobert-base-p1` |
| Random seed | 42 (split, inisialisasi bobot, dan setiap fold CV); 42/123/2024 untuk variabilitas antar-*seed* |
| Panjang token maksimum | 96 |
| Ukuran batch | 32 |
| Epoch | 3 |

Batas bawah versi mengikuti sel `pip install` pada notebook. Untuk mereproduksi angka persis, pin versi hasil `pip freeze` pada mesin yang dipakai.

Split data dan inisialisasi bobot dikendalikan `random_state`, dan seluruh *fold* CV memakai `random_state=SEED`.

---

## 📊 Ringkasan Hasil

**Model utama:** IndoBERT Base + *Global Average Pooling* + Dense Head (64 → 16 → 3), *dropout* 0,05.

**Metrik pada 664 data uji (seed 42):**

| Metrik | Nilai |
|---|---|
| Akurasi | 74,85% |
| *Weighted*-F1 | 75,09% |
| Macro-F1 | 72,56% |
| ROC-AUC | 89,24% |
| PR-AUC | 81,16% |
| Latensi inferensi | 5,434 ± 0,007 ms/komentar (median 5 ulangan) |
| *Throughput* | 184,0 RPS |
| Ukuran model | 474,9 MB |
| Konsumsi RAM | 2.041,1 MB |

**Pembanding terbaik per metrik** (dilaporkan terpisah, tidak digabung):

| Metrik | Model | Nilai | Selisih |
|---|---|---|---|
| Akurasi | Logistic Regression | 71,69% | 3,16 poin |
| Macro-F1 | Linear SVM | 67,80% | 4,76 poin |

*Baseline* lain: Naive Bayes Macro-F1 53,53%, Random Forest 67,62% akurasi.

**Variabilitas antar-*seed* (3 *seed*):** akurasi 76,66% ± 2,07%; Macro-F1 74,44% ± 2,45%. Rentang antar-*seed* (74,85–78,92%) lebih lebar daripada selisih terhadap *baseline*, sehingga angka tunggal tidak cukup.

**Uji signifikansi (10 sub-sampel data uji):** Wilcoxon p = 0,160; uji-t berpasangan p = 0,154 — keduanya **tidak signifikan**. Keunggulan dilaporkan sebagai kecenderungan, bukan bukti statistik.

***Ablation study* (seed 42):** tanpa GAP Macro-F1 75,65%; tanpa *dropout* 78,11%; keduanya di atas konfigurasi penuh (72,56%).

**Dampak insiden VAR (regresi logistik, seluruh 3.316 komentar):**

| Kategori | *Odds Ratio* | Proporsi negatif | p |
|---|---|---|---|
| Offside VAR | 3,341× | 73,16% | < 0,0001 |
| *Penalty Check* | 2,282× | 65,09% | < 0,0001 |
| Gol dianulir | 1,746× | — | — |

**Stratified 5-Fold CV pada data latih** (Macro-F1, semua *fold* dilaporkan):
SVM 61,09% ± 1,26 — *fold*: 61,08 / 60,89 / 59,07 / 63,02 / 61,39
Logistic Regression 58,92% ± 1,72 — *fold*: 59,80 / 61,90 / 57,77 / 57,27 / 57,85
Naive Bayes 46,78% ± 1,64 — *fold*: 44,32 / 48,75 / 47,39 / 45,46 / 47,98

Nilai CV jauh di bawah nilai uji (SVM 67,80%), tanda angka uji masih optimistis meski *checkpoint* dipilih lewat set validasi terpisah.

---

## ⚠️ Limitasi

* Hanya tiga *seed* dan satu konfigurasi *hyperparameter*, tanpa *tuning* sistematis.
* Deteksi sarkasme lemah; model tertipu kalimat tuduhan implisit tanpa kata kasar.
* Kesepakatan anotasi manual hanya 46,00% (Cohen's Kappa 0,19 — *slight agreement*), sehingga sebagian label mengandung derau.
* Komentar melebihi 96 token terpotong; penghapusan angka menghilangkan keterangan menit.
* Evaluasi terbatas pada satu laga dan satu platform.
* CV model IndoBERT belum dijalankan (`RUN_CV_INDOBERT = False`); CV hanya mencakup *baseline* klasik pada data latih dan bebas *leakage* karena `TfidfVectorizer` di-*fit* di dalam setiap *fold*.

---

## 📄 Lisensi

Kode: MIT. Dataset komentar: konten publik YouTube, dipakai untuk kepentingan penelitian.

## ✉️ Kontak

Syahid Ahmad Yasin — `15241007@bsi.ac.id`
Program Studi Informatika, Universitas Bina Sarana Informatika.

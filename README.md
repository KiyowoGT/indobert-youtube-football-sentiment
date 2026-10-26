# Analisis Sentimen Komentar YouTube Laga Portugal vs Kroasia Menggunakan IndoBERT

Repositori ini memuat kode validasi eksperimen dan dataset komentar YouTube berbahasa Indonesia pada laga Portugal melawan Kroasia, dengan fokus klasifikasi sentimen berbasis arsitektur *Transfer Learning* IndoBERT serta pemodelan regresi dampak insiden VAR (*Video Assistant Referee*).

---

## 📁 Struktur Repositori

```text
.
├── README.md
├── Validasi_Metode_4Paper_ke_Dataset_YouTube.ipynb
├── train_dataset_80.csv
└── test_dataset_20.csv
```

* **`Validasi_Metode_4Paper_ke_Dataset_YouTube.ipynb`** : Notebook utama validasi 4 paper acuan yang lolos pada dataset YouTube (lengkap dengan output cell evaluasi, grafik, dan tabel regresi).
* **`train_dataset_80.csv`** : Data latih 80% (2.652 baris) dengan kolom `comment`, `sentiment`, `sarkasme`.
* **`test_dataset_20.csv`** : Data uji 20% (664 baris) dengan kolom `comment`, `sentiment`, `sarkasme`.

---

## 🚀 Panduan Menjalankan Kode

### Opsi 1: Menjalankan di Lokal / VS Code / Jupyter Lab (Disarankan)
1. *Clone* repositori ini:
   ```bash
   git clone https://github.com/<username>/<nama-repo>.git
   cd <nama-repo>
   ```
2. Pastikan file **`train_dataset_80.csv`** dan **`test_dataset_20.csv`** berada di **folder yang sama** dengan notebook `Validasi_Metode_4Paper_ke_Dataset_YouTube.ipynb`.
3. Buka notebook dan jalankan (*Run All*). Kode otomatis membaca dataset dari direktori aktif:
   ```python
   df_train = pd.read_csv('train_dataset_80.csv')
   df_test = pd.read_csv('test_dataset_20.csv')
   ```

### Opsi 2: Menjalankan di Google Colab Menggunakan Google Drive
Jika menggunakan Google Colab dan ingin menyimpan dataset di Google Drive:
1. Buat folder bernama **`dataset`** langsung di direktori utama Drive Anda:
   ```text
   Drive Saya (MyDrive)/
   └── dataset/
       ├── train_dataset_80.csv
       └── test_dataset_20.csv
   ```
2. Upload kedua file CSV ke folder `dataset/` tersebut.
3. Jalankan notebook di Colab. Kode akan otomatis melakukan mount Google Drive dan membaca data dari `/content/drive/MyDrive/dataset/`.

---

## 📊 Ringkasan Hasil Eksperimen

* **Model Utama:** IndoBERT Base (`indobenchmark/indobert-base-p1`) + Global Average Pooling + Dense Head (64 -> 16 -> 3).
* **Metrik Evaluasi pada 664 Data Uji:**
  * Akurasi: **78,77%**
  * Weighted-F1: **78,67%**
  * Macro-F1: **76,70%**
  * ROC-AUC: **89,90%**
  * PR-AUC: **80,65%**
  * Latensi Inferensi: **5,638 ms/komentar** (177,4 RPS)
* **Dampak Insiden VAR (Regresi Logistik):**
  * Offside VAR: **Odds Ratio 3,22x** lipat lebih berpeluang memicu sentimen negatif ($p < 0,0001$).
  * Penalty Check: **Odds Ratio 2,23x** lipat lebih berpeluang memicu sentimen negatif ($p < 0,0001$).

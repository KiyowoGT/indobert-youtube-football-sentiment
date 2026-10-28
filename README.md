# Dataset Komentar YouTube Portugal vs Kroasia untuk Analisis Sentimen

Dataset komentar YouTube berbahasa Indonesia pada laga Portugal melawan Kroasia, disertai kode replikasi empat paper acuan. Dataset dipakai untuk klasifikasi sentimen tiga kelas (`negatif`, `netral`, `positif`) dengan *Transfer Learning* IndoBERT, serta pemodelan regresi dampak insiden VAR (*Video Assistant Referee*).

Seluruh angka di naskah dihasilkan dari dataset ini, bukan dihitung manual.

---

## 📁 Isi Repositori

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
│       └── validasi_label_manual_terisi.csv      # 150 baris, sudah terisi
├── validation/
│   └── Validasi_Metode_4Paper_ke_Dataset_YouTube.ipynb
└── replication/
    ├── Replikasi_Lolos/                          # 4 paper yang berhasil direplikasi
    └── Replikasi_Tidak_Lolos/                    # 9 paper yang gagal direplikasi
```

---

## 📊 Rincian Dataset

### `data/raw/youtube_comments.csv`

Keluaran apa adanya dari YouTube Data API v3, sebelum penyaringan apa pun.

| Kolom | Tipe | Keterangan |
|---|---|---|
| `author` | teks | Nama kanal pengomentar |
| `comment` | teks | Isi komentar mentah |
| `likes` | angka | Jumlah suka |
| `is_reply` | boolean | Balasan komentar atau bukan |

5.460 baris.

### `data/processed/dataset_komentar_indonesia_bersih.csv`

Hasil tahap penyaringan bahasa dua sisi. Berkas ini masih memuat kolom anotasi.

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id_komentar` | teks | Kunci `ID_00001` dan seterusnya |
| `author` | teks | Nama kanal |
| `clean_comment` | teks | Komentar setelah normalisasi |
| `likes` | angka | Jumlah suka |
| `sentimen` | teks | `negatif` / `netral` / `positif` |
| `sarkasme` | teks | `ya` / `tidak` |
| `tugas_anotator` | teks | Nama anotator |

2.412 baris.

### `data/processed/dataset_youtube_labeled_all.csv`

Korpus penuh setelah seluruh penyaringan dan pelabelan. Hanya tiga kolom: `comment`, `sentiment`, `sarkasme` — metadata kanal dan suka dilepas di tahap ini.

3.316 baris — inilah korpus yang dipakai penelitian.

### `data/processed/train_dataset_80.csv` dan `test_dataset_20.csv`

Split terstratifikasi 80:20 dari korpus penuh, `random_state=42`. Kolom `comment`, `sentiment`, `sarkasme`. 2.652 baris latih dan 664 baris uji, tanpa satu pun komentar identik yang menyeberang.

### `data/processed/dataset_dibuang.csv`

Komentar yang gugur saat penyaringan. Kolom `cleaned_text` dan `alasan`. 2.144 baris: 948 kurang dari 3 kata, 639 bukan bahasa Indonesia, 515 di luar topik, 42 duplikat. Berguna untuk menelusuri kalau ada yang mempertanyakan jumlah akhir.

### `data/processed/validasi_label_manual.csv` dan `validasi_label_manual_terisi.csv`

150 sampel uji untuk mengukur kesepakatan anotasi. Kolom `label_otomatis` sudah terisi di keduanya; `label_manual` kosong pada versi pertama dan terisi pada versi kedua. Hasil ukur: kesepakatan 46,00%, Cohen's Kappa 0,19 (*slight agreement*).

---

## 🚀 Cara Memakai Dataset

### Memuat di pandas

```python
import pandas as pd

train = pd.read_csv('data/processed/train_dataset_80.csv')
test = pd.read_csv('data/processed/test_dataset_20.csv')

print(train.shape)   # (2652, 3)
print(train['sentiment'].value_counts())
```

### Memuat di Google Colab

```python
from google.colab import drive
drive.mount('/content/drive')
# unggah kedua CSV ke Drive Saya/dataset/ lebih dulu
train = pd.read_csv('/content/drive/MyDrive/dataset/train_dataset_80.csv')
```

### Mengukur ulang kesepakatan anotasi

```python
from sklearn.metrics import cohen_kappa_score

df = pd.read_csv('data/processed/validasi_label_manual_terisi.csv').dropna(subset=['label_manual'])
label = ['negatif', 'netral', 'positif']
print('kesepakatan:', (df.label_manual == df.label_otomatis).mean() * 100)
print('kappa:', cohen_kappa_score(df.label_manual, df.label_otomatis, labels=label))
```

---

## ⚙️ Lingkungan

```bash
python -m venv .venv && source .venv/bin/activate
pip install "transformers>=4.30.0" "torch>=2.0.0" "scikit-learn>=1.2.0" \
            "statsmodels>=0.14.0" seaborn matplotlib scipy psutil pandas numpy jupyterlab
jupyter lab            # lalu buka validation/ atau replication/ dari peramban
```

| Komponen | Nilai |
|---|---|
| Python | 3.10 (runtime Google Colab) |
| Perangkat keras | GPU NVIDIA CUDA (Tesla T4, Colab) |
| Model dasar | `indobenchmark/indobert-base-p1` |
| Random seed | 42 (split, bobot awal, setiap fold CV) |
| Panjang token maksimum | 96 |
| Ukuran batch | 32 |
| Epoch | 3 |

Batas bawah versi mengikuti sel `pip install` pada notebook. Untuk mereproduksi angka persis, pin versi hasil `pip freeze` pada mesin yang dipakai.

---

## 🧪 Kode Replikasi

`replication/Replikasi_Lolos/` memuat empat paper yang berhasil direplikasi pada dataset aslinya, dan `replication/Replikasi_Tidak_Lolos/` sembilan yang gagal. Folder ini menjadi dasar pemilihan metode: hanya arsitektur yang lolos yang dipindahkan ke korpus YouTube ini.

## 📌 Catatan

`validation/Validasi_Metode_4Paper_ke_Dataset_YouTube.ipynb` adalah notebook eksperimen. Seluruh angka di naskah berasal dari berkas ini. Isinya melatih IndoBERT, empat *baseline* klasik, dekomposisi matriks konfusi, ROC/PR, Stratified 5-Fold CV, uji Wilcoxon dan uji-t berpasangan, pengukuran latensi, serta regresi logistik insiden VAR. 22 sel, 11 berjalan, 6 gambar.

Jalankan dari akar repositori, karena memakai path relatif dan membaca `data/processed/train_dataset_80.csv` serta `test_dataset_20.csv`.

### Versi revisi tidak disertakan

Ada notebook revisi yang lebih baru, tetapi **tidak** ada di repo. Versi itu memperbaiki aturan kata kunci kategori VAR, sehingga angkanya bergeser dari naskah:

| Metrik (data uji) | Naskah (notebook di repo) | Versi revisi |
|---|---|---|
| Akurasi | 74,85% | 78,77% |
| Macro-F1 | 72,56% | 76,70% |
| ROC-AUC | 89,24% | 89,90% |
| Latensi | 5,434 ms | 5,03 ms |
| *Seed* | 42, 123, 2024 | 42, 43, 44 |
| Uji Wilcoxon / uji-t | ada, p = 0,160 / 0,154 | **tidak ada** |

Selain Tabel 1, Tabel 2, dan koefisien OLS yang ikut bergeser, versi revisi juga memakai *seed* lain (42/43/44) dan menghapus uji Wilcoxon serta uji-t. Kalau nanti dipakai, abstrak, 3.1, dan kesimpulan harus diperbarui lebih dulu.

## 📄 Lisensi

Kode: MIT. Dataset komentar: konten publik YouTube, dipakai untuk kepentingan penelitian.

## ✉️ Kontak

Syahid Ahmad Yasin — `15241007@bsi.ac.id`
Program Studi Informatika, Universitas Bina Sarana Informatika.

# Panduan Unggah ke Git

Checklist ini mengikuti Pertemuan 14 (Review Codebase) dan Pertemuan 13 (Validasi Metrik). Tujuannya: dosen bisa membuka repo, menjalankan kode, dan mendapatkan angka yang sama dengan naskah.

## Prasyarat: jalankan dulu, jangan unggah dulu

Sebelum `git push` pertama, buktikan repo ini jalan dari nol di folder bersih.

```bash
# di salinan bersih repo, bukan di folder kerja
python -m venv .venv && source .venv/bin/activate
pip install "transformers>=4.30.0" "torch>=2.0.0" "scikit-learn>=1.2.0" \
            "statsmodels>=0.14.0" seaborn matplotlib scipy psutil pandas numpy jupyterlab
jupyter lab validation/
```

Jalankan notebook `01` sampai `02` berurutan. Kalau ada yang gagal, repo belum layak unggah.

## 1. Yang WAJIB diunggah

Semua sudah disiapkan dalam struktur di README.

| Berkas | Alasan |
|---|---|
| `README.md` | Panduan masuk, sudah disamakan dengan angka naskah |
| `PANDUAN_UNGGAH_GIT.md` | Checklist ini |
| `.gitignore` | Mencegah `.venv`, model, dan berkas kerja ikut |
| `validation/01_Preprocessing_Penyortiran_Split80_20.ipynb` | Jejak penyaringan dan split |
| `validation/02_Validasi_Metode_4Paper_ke_Dataset_YouTube.ipynb` | Notebook utama, **wajib versi tereksekusi** |
| `data/raw/youtube_comments.csv` | Sumber mentah 5.460 komentar |
| `data/processed/dataset_youtube_labeled_all.csv` | Korpus berlabel 3.316 |
| `data/processed/dataset_dibuang.csv` | Bukti 2.144 komentar tereliminasi + alasannya |
| `data/processed/dataset_komentar_indonesia_bersih.csv` | Keluaran tahap saring bahasa |
| `data/processed/train_dataset_80.csv` | 2.652 baris latih |
| `data/processed/test_dataset_20.csv` | 664 baris uji |
| `data/processed/validasi_label_manual_terisi.csv` | Dasar hitung Kappa 0,19 |
| `replication/` | Bukti replikasi 13 paper acuan, dasar pemilihan metode |

Repo ini sengaja **hanya** memuat dataset, kode validasi, kode replikasi, dan berkas `.md`. Skrip penyusun naskah, kerangka templat, arsip, `LICENSE`, dan `requirements.txt` tidak diunggah.

## 2. Yang JANGAN diunggah

Sudah ditangani `.gitignore`, pastikan tetap begitu.

| Berkas | Alasan |
|---|---|
| `.venv/` | Ribuan berkas, bisa dibuat ulang lewat `requirements.txt` |
| `__pycache__/`, `*.pyc` | Artefak |
| `.ipynb_checkpoints/` | Duplikat notebook |
| `*.docx`, `*.pdf` | Naskah dikumpulkan terpisah, bukan lewat repo |
| `build_jati/`, `build_jati_clean/`, `build_jati_compliant/`, `build_jati_refined/` | Varian lama, hanya `build_jati_rigorous` yang dipakai |
| `_hf/`, `_pylibs/` | Cache pustaka |
| `_m3/`, `_mat2/`, `_materi/`, `_t/`, `_x/` | Materi kuliah dan folder kerja sementara |
| `*.pt`, `*.bin`, `*.safetensors`, `models/` | Bobot model, ratusan MB |
| `page_rendered-*.png`, `pg-*.png`, `figs_nb*/` | Pratinjau halaman, bukan keluaran resmi |
| `jati_media/` | Salinan gambar lama |
| duplikat di root | Salinan resmi sudah ada di `data/`, `notebooks/`, `scripts/` |

## 3. Berkas yang harus diputuskan

| Berkas | Keputusan |
|---|---|
| `Validasi_Lama_Regex_Diperbaiki.ipynb` | Versi lama (10 sel tereksekusi). **Jangan unggah** agar tidak bersaing dengan notebook `02` |
| `validasi_label_manual_terisi.xlsx` | Opsional. Unggah kalau dosen minta bentuk Excel |
| `dataset_1000_siap_anotasi.csv` | Opsional, tidak dirujuk naskah |
| `figs_nb3_*.png`, `page_rendered-*.png`, `pg-6.png` | Pratinjau halaman, tidak perlu |

## 4. Perintah unggah

```bash
cd "<folder repo>"
git init
git add .
git status                    # periksa daftar, pastikan tidak ada .venv atau model
git commit -m "Dataset, notebook eksperimen, dan skrip penyusun naskah"
git branch -M main
git remote add origin https://github.com/KiyowoGT/indobert-youtube-football-sentiment
git push -u origin main
```

Repo tujuan di README: `https://github.com/KiyowoGT/indobert-youtube-football-sentiment`

## 5. Periksa hasil unggahan

1. Buka repo di peramban tanpa login.
2. `README.md` tampil dan strukturnya cocok dengan isi repo.
3. Notebook `02` terbuka **dengan output dan gambar**, bukan kosong.
4. Unduh ZIP repo, ekstrak ke folder baru, jalankan ulang sesuai README. Kalau berhasil, repo sudah benar.

## 6. Sebelum dikumpulkan ke dosen

- [ ] Semua angka README cocok dengan naskah dan keluaran notebook
- [ ] Notebook `02` versi tereksekusi (11 sel berkode, 6 gambar), bukan yang 10 sel
- [ ] `RUN_CV_INDOBERT` masih `False` — sebutkan di catatan kalau dosen menanyakan CV model utama
- [ ] Tautan repo di subbab 2.1 naskah mengarah ke repo yang benar-benar ada
- [ ] Tidak ada kunci API, token, atau path pribadi yang tertinggal
- [ ] Riwayat commit bersih, tanpa nama berkas sementara

## 7. Setelah terunggah

Tiga hal ini masih terbuka dan akan ditagih dosen:

1. **CV model IndoBERT** — Pertemuan 13 menuntut pelatihan berjalan di setiap fold. Sekarang hanya baseline klasik.
2. **Pseudocode** — Pertemuan 04 meminta bila ada modifikasi alur; GAP + dense head termasuk.
3. **Versi pustaka persis** — `requirements.txt` masih batas bawah (`>=`). Ganti dengan hasil `pip freeze` dari mesin yang benar-benar dipakai.

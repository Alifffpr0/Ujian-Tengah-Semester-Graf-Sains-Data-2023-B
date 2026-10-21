# UTS Graf Sains Data — Analisis Graf dan Rekomendasi KNN

**Mata kuliah:** Graf Sains Data  
**Nama:** Muhammad Alif Mahbubi  
**NIM:** 23031554031  
**Kelas:** 2023B

Proyek ini menggunakan dataset interaksi e-commerce RetailRocket untuk membangun graf User–Item di Neo4j, menganalisis User Projection dan Item Projection, menghitung kemiripan antar-pengguna dengan cosine similarity, lalu membuat rekomendasi item menggunakan K-Nearest Neighbors (KNN). Hasil rekomendasi dievaluasi menggunakan Hit@5 dan NDCG@5.

---

## Daftar Isi

- [1. Tujuan Proyek](#1-tujuan-proyek)
- [2. Teknologi yang Digunakan](#2-teknologi-yang-digunakan)
- [3. Dataset](#3-dataset)
- [4. Struktur Folder](#4-struktur-folder)
- [5. Persiapan di Windows PowerShell](#5-persiapan-di-windows-powershell)
- [6. Menyiapkan Neo4j](#6-menyiapkan-neo4j)
- [7. Menjalankan Notebook](#7-menjalankan-notebook)
- [8. Alur Analisis](#8-alur-analisis)
- [9. Ringkasan Hasil pada Notebook](#9-ringkasan-hasil-pada-notebook)
- [10. Mengatasi Masalah Umum](#10-mengatasi-masalah-umum)
- [11. Mengunggah Proyek ke GitHub](#11-mengunggah-proyek-ke-github)

## 1. Tujuan Proyek

Proyek ini dikerjakan untuk menerapkan konsep graf pada data interaksi pengguna dan item. Tahapan utamanya adalah:

1. Melakukan pemeriksaan data, konversi timestamp, dan pemisahan periode training dan testing.
2. Membentuk graf interaksi User–Item di Neo4j.
3. Membentuk matriks VSM User–Item dan VSM Item–User dalam format sparse.
4. Membentuk serta mengeksplorasi User Projection dan Item Projection.
5. Membandingkan kedua proyeksi berdasarkan jumlah edge, density, dan waktu pembentukan.
6. Menghitung cosine similarity antar-pengguna pada User Projection.
7. Menyimpan lima hubungan similarity teratas ke Neo4j dan memeriksanya dengan Cypher.
8. Membuat rekomendasi item dengan KNN untuk seorang User yang dipilih secara acak dan dapat direproduksi.
9. Mengevaluasi rekomendasi dengan Hit@5 dan NDCG@5.
10. Menampilkan graf sebelum dan sesudah hubungan similarity ditambahkan.

## 2. Library yang Digunakan

- **Python** untuk pengolahan data dan perhitungan.
- **Jupyter Notebook dan Visual Studio Code** untuk menjalankan kode secara bertahap.
- **Pandas dan NumPy** untuk pengolahan serta perhitungan data.
- **SciPy sparse matrix** untuk menyimpan matriks User–Item dengan lebih hemat memori.
- **scikit-learn** untuk perhitungan cosine similarity.
- **NetworkX dan Matplotlib** untuk membentuk serta memvisualisasikan graf.
- **Neo4j** untuk menyimpan graf dan menjalankan query Cypher.
- **tqdm** untuk menampilkan progres ketika data diimpor dalam batch.

## 3. Dataset

### Sumber

Dataset yang digunakan adalah **RetailRocket e-commerce events dataset**, yang berisi catatan interaksi pengguna terhadap item di situs e-commerce. Salah satu sumber publik dataset ini adalah [RetailRocket E-commerce Dataset di Kaggle](https://www.kaggle.com/retailrocket/ecommerce-dataset).

Notebook ini hanya memakai file **`events.csv`**. File lain dari paket dataset, seperti data properti item dan pohon kategori, tidak digunakan dalam alur analisis pada notebook ini.

### Kolom yang digunakan

| Kolom | Keterangan |
|---|---|
| `timestamp` | Waktu kejadian dalam format Unix milidetik. |
| `visitorid` | Identitas pengguna. |
| `event` | Jenis interaksi: `view`, `addtocart`, atau `transaction`. |
| `itemid` | Identitas item. |
| `transactionid` | Identitas transaksi; dapat kosong untuk interaksi selain transaksi. |

Pada notebook yang digunakan saat dokumentasi ini dibuat, dataset utama terbaca sebanyak **2.756.101 baris dan 5 kolom**. Rentang waktunya dimulai pada Mei 2015 dan berakhir pada September 2015. Nilai `timestamp` diubah menjadi waktu UTC sebelum data dipisahkan.

### Pembagian training dan testing

| Pembagian | Periode UTC | Penggunaan |
|---|---|---|
| **P1 — Training** | 5–7 Juli 2015 | Membentuk graf, matriks User–Item, proyeksi, dan rekomendasi. |
| **P2 — Testing** | 16–31 Juli 2015 | Menjadi data aktual untuk mengevaluasi rekomendasi. |

Periode dibuat dengan batas akhir eksklusif: P1 menggunakan interval `[2015-07-05, 2015-07-08)`, sedangkan P2 menggunakan `[2015-07-16, 2015-08-01)`. Notebook membuat dua file hasil pemisahan ini secara otomatis.

Ringkasan hasil filtering yang tersimpan pada notebook:

| Data | Jumlah interaksi | User unik | Item unik |
|---|---:|---:|---:|
| P1 — Training | 62.491 | 34.436 | 29.766 |
| P2 — Testing | 364.206 | 199.899 | 88.699 |

> **Catatan:** simpan file mentah secara lokal di `data/raw/events.csv`. Ukuran file dataset cukup besar sehingga tidak perlu dimasukkan ke GitHub kecuali memang diminta oleh dosen. Ikuti ketentuan penggunaan dan distribusi dari sumber dataset.

## 4. Struktur Folder

Struktur folder yang disarankan:

```text
uts-graf-sains-data/
├── README.md
├── requirements.txt
├── .gitignore
├── UTS_Graf_Sains_Data_rapi.ipynb
└── data/
    ├── raw/
    │   └── events.csv
    └── processed/
        ├── events_P1_train.csv
        └── events_P2_test.csv
```

`events.csv` adalah data mentah. File `events_P1_train.csv` dan `events_P2_test.csv` dibuat otomatis ketika bagian pemisahan data pada notebook dijalankan. Jika nama notebook di repositori berbeda, gunakan nama file yang benar ketika membukanya di VS Code.

## 5. Persiapan di Windows PowerShell

Langkah-langkah berikut ditulis untuk Windows dan PowerShell. Jalankan perintah satu per satu.

### 5.1. Buka folder proyek

Jika repositori sudah dibuat di GitHub, kloning repositorinya terlebih dahulu. Ganti URL contoh dengan URL repositori milik sendiri.

```powershell
git clone https://github.com/USERNAME/NAMA-REPOSITORY.git
Set-Location .\NAMA-REPOSITORY
```

Jika folder proyek sudah tersedia di komputer, langsung masuk ke folder tersebut, misalnya:

```powershell
Set-Location "C:\Users\user\gsd_uts"
```

Sesuaikan alamat folder dengan lokasi proyek di komputer masing-masing.

### 5.2. Buat folder data

```powershell
New-Item -ItemType Directory -Force -Path "data\raw", "data\processed" | Out-Null
```

### 5.3. Pastikan Python tersedia

Contoh berikut menggunakan Python 3.13, sesuai lingkungan pengerjaan. Jika versi tersebut tidak terpasang, gunakan versi Python yang tersedia dan kompatibel dengan library yang dibutuhkan.

```powershell
py -3.13 --version
```

Jika perintah di atas tidak tersedia, periksa instalasi Python dengan:

```powershell
py --list
```

### 5.4. Buat dan aktifkan virtual environment

Virtual environment memisahkan library proyek ini dari instalasi Python yang lain.

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Jika PowerShell menolak menjalankan skrip aktivasi karena Execution Policy, izinkan hanya untuk sesi PowerShell saat ini, kemudian aktifkan kembali:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

Jika berhasil, nama `(.venv)` biasanya muncul di awal baris PowerShell.

### 5.5. Buat `requirements.txt`

Jika file ini belum tersedia dalam repositori, buat menggunakan perintah berikut:

```powershell
@'
numpy
pandas
scipy
scikit-learn
networkx
matplotlib
neo4j
tqdm
ipykernel
jupyter
'@ | Set-Content -Encoding utf8 requirements.txt
```

Kemudian perbarui pip dan instal semua kebutuhan proyek:

```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Notebook juga memiliki cell `%pip install neo4j`. Jika library Neo4j belum tersedia pada kernel yang sedang digunakan, cell tersebut dapat dijalankan untuk memasang driver.

### 5.6. Letakkan dataset

Unduh paket RetailRocket dari sumber yang digunakan, lalu tempatkan file `events.csv` di lokasi berikut:

```text
data/raw/events.csv
```

Pastikan nama file dan folder tepat. Notebook menggunakan path relatif `data/raw/events.csv`, sehingga file perlu berada di lokasi tersebut relatif terhadap folder utama proyek.

### 5.7. Buka notebook di Visual Studio Code

```powershell
code .
```

Di VS Code:

1. Pastikan ekstensi **Python** dan **Jupyter** sudah terpasang.
2. Buka `UTS_Graf_Sains_Data_rapi.ipynb`.
3. Pilih kernel Python dari virtual environment `.venv`.
4. Jalankan cell secara berurutan dari atas ke bawah ketika menyiapkan lingkungan untuk pertama kali.

Jika perintah `code .` tidak dikenali, buka VS Code secara manual lalu pilih **File → Open Folder** dan pilih folder proyek.

## 6. Menyiapkan Neo4j

Notebook berkomunikasi dengan database lokal Neo4j melalui driver Python.

### 6.1. Menjalankan database

1. Pasang dan buka Neo4j Desktop.
2. Buat atau buka local DBMS yang digunakan untuk tugas ini.
3. Pastikan database berjalan (**Running**).
4. Pastikan nama database yang digunakan notebook adalah `neo4j`, atau sesuaikan parameter `database` pada seluruh pemanggilan `driver.session()` jika menggunakan nama lain.
5. Periksa alamat Bolt dan port yang ditampilkan oleh Neo4j Desktop. Konfigurasi notebook saat ini menggunakan URI `neo4j://127.0.0.1:7687` dan username `neo4j`.

### 6.2. Menghubungkan Python ke Neo4j

Notebook meminta password menggunakan `getpass()`, sehingga password tidak perlu ditulis langsung di kode.

```python
URI = "neo4j://127.0.0.1:7687"
USERNAME = "neo4j"
PASSWORD = getpass("Masukkan password Neo4j: ")
```

Jangan menyimpan password ke notebook, README, atau repositori GitHub. Jika muncul error `Unable to retrieve routing information`, pastikan database sedang berjalan dan alamat/port sesuai konfigurasi Neo4j Desktop. Untuk instance lokal tunggal, koneksi langsung `bolt://127.0.0.1:7687` dapat dicoba jika alamat Bolt pada Neo4j Desktop memang menggunakan port tersebut.

### 6.3. Perhatian sebelum impor data

Proses impor pada notebook ditujukan untuk database yang masih kosong. Notebook memeriksa jumlah node dan relationship sebelum impor dan menghentikan proses jika database sudah berisi data. Langkah ini mencegah relationship interaksi terduplikasi.

**Jangan menjalankan bagian impor P1 berulang kali pada database yang sama.** Fungsi impor menggunakan `CREATE` untuk relationship interaksi, sehingga impor berulang dapat menambah relationship duplikat. Setelah data berhasil diimpor, ketika membuka kernel Python baru, siapkan kembali variabel Python dari awal tetapi lewati cell yang memaksa database kosong dan cell impor apabila data Neo4j yang benar masih tersedia.

Jika ingin mengulang proses impor dari awal, gunakan database tugas yang baru dan kosong atau lakukan reset hanya setelah memastikan bahwa data yang akan dihapus memang tidak dibutuhkan. Jangan mengosongkan database yang berisi proyek lain.

## 7. Menjalankan Notebook

Untuk eksekusi pertama kali, ikuti alur berikut secara berurutan.

1. **Persiapan library dan dataset:** memasang library, memeriksa `events.csv`, membaca data, dan mengubah `timestamp` menjadi waktu UTC.
2. **Pemisahan data:** membuat `train_df` untuk P1 dan `test_df` untuk P2, lalu menyimpan kedua file CSV ke `data/processed/`.
3. **Koneksi Neo4j:** mengatur driver dan memeriksa koneksi database.
4. **Impor graf:** membuat constraint ID, lalu mengimpor interaksi P1 secara batch dengan ukuran 5.000 baris per batch.
5. **Pemeriksaan graf:** memeriksa jumlah node dan relationship serta mengambil pasangan User–Item dari Neo4j.
6. **VSM User dan Item:** membentuk matriks biner sparse User–Item dan Item–User.
7. **Proyeksi graf:** membentuk User Projection dan Item Projection, memvisualisasikan sampel graf, dan membandingkan karakteristik kedua proyeksi.
8. **Similarity untuk soal graf:** memilih User acuan dengan `SEED = 42`, menghitung lima tetangga terdekat, menyimpan nilai cosine similarity sebagai relationship `SIMILAR_USER`, mengambilnya kembali dengan Cypher, lalu membandingkan nilai Python dan Neo4j.
9. **Analytics rekomendasi:** memilih `analytics_user` secara terpisah menggunakan `SEED_ANALYTICS = 42`, menyusun rekomendasi untuk `K = 5`, dan menampilkan tiga item teratas.
10. **Evaluasi:** membangun ground truth dari item yang muncul pada P2 tetapi tidak ada pada riwayat P1, lalu menghitung Hit@5 dan NDCG@5.

### Eksekusi berikutnya setelah data sudah diimpor

Jika kernel direstart, seluruh variabel Python akan hilang, tetapi data Neo4j tetap tersimpan. Jalankan kembali cell pembacaan dataset dan persiapan variabel Python. Jangan menjalankan cell pemeriksaan yang mensyaratkan database kosong serta cell impor P1 lagi jika database sudah terisi. Lanjutkan dari cell pemeriksaan dan pembacaan graf yang sudah ada.

Pada notebook, variabel `user_ids` digunakan untuk menjaga pemetaan antara ID User dan indeks matriks. Jangan menimpa variabel itu dengan daftar User untuk visualisasi. Gunakan nama lain, misalnya `graph_user_ids`, agar dimensi `A_user_proj` dan pemetaan indeks tetap sesuai.

## 8. Alur Analisis

### 8.1. Eksplorasi dan pemisahan data

Data diperiksa melalui ukuran dataset, nama kolom, rentang waktu, jumlah interaksi, jumlah User dan Item unik, serta distribusi jenis event. Kolom `timestamp` dikonversi dari Unix milidetik ke datetime UTC.

### 8.2. Graf User–Item di Neo4j

Node utama yang dibangun adalah:

- `(:Visitor {visitorId})` untuk pengguna.
- `(:Item {itemId})` untuk item.

Jenis relationship interaksi:

- `(:Visitor)-[:VIEWED {timestamp}]->(:Item)` untuk event `view`.
- `(:Visitor)-[:ADDED_TO_CART {timestamp}]->(:Item)` untuk event `addtocart`.
- `(:Visitor)-[:BOUGHT {timestamp, transactionId}]->(:Item)` untuk event `transaction`.

Node dibuat dengan `MERGE` berdasarkan ID, sedangkan relationship merepresentasikan kejadian interaksi dari baris data P1.

### 8.3. VSM dan proyeksi

Matriks VSM User–Item menggunakan baris untuk User dan kolom untuk Item. Nilai `1` menunjukkan bahwa pasangan User–Item memiliki interaksi pada data training. Format sparse CSR dipakai untuk menghemat memori.

- **User Projection:** dua User terhubung apabila keduanya memiliki interaksi terhadap Item yang sama.
- **Item Projection:** dua Item terhubung apabila keduanya pernah diinteraksikan oleh User yang sama.

Notebook membandingkan jumlah node, kemungkinan pasangan, kontribusi pasangan, edge unik, density, dan waktu pembentukan kedua proyeksi. Waktu komputasi dapat berbeda antarkomputer.

### 8.4. Similarity dan KNN

Cosine similarity dihitung pada baris-baris matriks User Projection. User acuan tidak dianggap sebagai tetangganya sendiri. Lima User dengan similarity positif tertinggi dipilih sebagai tetangga KNN; jika skor sama, ID User digunakan sebagai penentu urutan.

Lima nilai similarity teratas disimpan ke Neo4j sebagai relationship `SIMILAR_USER`, dengan properti `score`, `method`, dan `projection`. Hasilnya dibaca kembali dengan Cypher dan dibandingkan dengan hasil Python.

### 8.5. Rekomendasi item

Untuk Analytics, User dipilih secara acak dengan seed tetap dari kandidat yang memenuhi syarat. Sistem menggunakan lima tetangga (`K = 5`) untuk mengumpulkan item dari interaksi training. Skor item dihitung dengan menjumlahkan similarity tetangga yang pernah berinteraksi dengan item tersebut. Item yang sudah ada dalam riwayat User target dikeluarkan, kemudian kandidat diurutkan menurut skor rekomendasi, jumlah tetangga pendukung, dan ID item.

### 8.6. Evaluasi Hit@5 dan NDCG@5

- **Hit@5:** bernilai `1` jika setidaknya satu dari lima rekomendasi terdapat pada ground truth; jika tidak ada, nilainya `0`.
- **NDCG@5:** menilai relevansi dan posisi item relevan dalam lima rekomendasi teratas. Nilai mendekati `1` menunjukkan urutan yang lebih baik, sedangkan nilai `0` berarti tidak ada item relevan dalam Top-5 untuk kasus yang sedang diuji.

Ground truth pada notebook didefinisikan sebagai item unik yang muncul pada P2 tetapi belum ada di riwayat P1. Jika ground truth kosong, evaluasi untuk User tersebut harus diperiksa kembali dan tidak boleh ditafsirkan sebagai hasil evaluasi yang valid.

## 9. Ringkasan Hasil pada Notebook

Angka di bawah ini adalah hasil yang tersimpan pada notebook saat dokumentasi dibuat. Waktu eksekusi dapat berbeda tergantung perangkat.

| Hasil | Nilai yang tersimpan |
|---|---:|
| Interaksi training P1 | 62.491 |
| Interaksi testing P2 | 364.206 |
| Node `Visitor` | 34.436 |
| Node `Item` | 29.766 |
| Relationship interaksi P1 | 62.491 |
| Edge unik User Projection | 69.756 |
| Edge unik Item Projection | 212.423 |
| User acuan untuk query similarity graf | `731514` |
| User Analytics yang terpilih | `139754` |

Pada output Analytics yang tersimpan, tiga rekomendasi teratas adalah:

| Peringkat | Item | Skor rekomendasi | Tetangga pendukung |
|---:|---:|---:|---:|
| 1 | `37773` | 0.885957 | 2 |
| 2 | `217078` | 0.716855 | 2 |
| 3 | `313852` | 0.666667 | 1 |

Dalam output evaluasi yang tersimpan, item relevan dari P2 tidak muncul pada lima rekomendasi teratas, sehingga **Hit@5 = 0** dan **NDCG@5 = 0** untuk User yang diuji. Hasil satu User ini tidak cukup untuk menyimpulkan performa model secara keseluruhan; penilaian umum perlu dilakukan pada lebih banyak User.

## 10. Mengatasi Masalah Umum

### `FileNotFoundError: data/raw/events.csv`

Pastikan `events.csv` berada di `data/raw/events.csv` relatif terhadap folder utama proyek. Periksa juga ejaan nama file.

### `ServiceUnavailable: Unable to retrieve routing information`

Pastikan Neo4j Desktop sudah menjalankan database, alamat/port Bolt benar, dan database yang digunakan sesuai. Jika konfigurasi lokal mendukung koneksi langsung, coba URI `bolt://127.0.0.1:7687`.

### `RuntimeError: Database tidak kosong`

Ini merupakan pengaman dari notebook. Artinya database sudah memiliki node atau relationship, sehingga cell impor berhenti agar tidak menambahkan relationship interaksi lagi. Jangan menjalankan impor berulang kali pada database yang sama.

### Jumlah relationship menjadi berlipat

Kemungkinan cell impor P1 dijalankan berulang kali. Periksa database sebelum mengimpor dan gunakan database baru yang kosong hanya jika memang ingin membangun ulang graf dari awal.

### `NameError: name ... is not defined`

Variabel yang diperlukan belum dibuat pada kernel saat ini atau cell sebelumnya belum dijalankan. Jalankan cell dari tahap yang membuat variabel tersebut, secara berurutan.

### Ukuran `A_user_proj` tidak sesuai dengan `user_ids`

Jangan gunakan kembali nama `user_ids` untuk keperluan visualisasi. Variabel tersebut harus tetap berisi seluruh ID User sesuai urutan baris matriks. Untuk kebutuhan graf visualisasi, gunakan `graph_user_ids`.

### `ground_truth_items` belum didefinisikan

Jalankan cell persiapan evaluasi yang membentuk `top5_recommendations`, `test_items`, `training_history`, dan `ground_truth_items` sebelum menghitung Hit@5 dan NDCG@5.

### Ground truth kosong

User yang dipilih tidak memiliki item baru pada P2 dibandingkan riwayat P1. Evaluasi tidak dapat diinterpretasikan secara bermakna untuk User itu. Periksa kembali kriteria pemilihan User evaluasi.

## 11. Mengunggah Proyek ke GitHub

### 11.1. Periksa file sebelum diunggah

Buat `.gitignore` di folder utama proyek agar file lokal dan data besar tidak ikut diunggah secara tidak sengaja. Contoh:

```gitignore
.venv/
__pycache__/
.ipynb_checkpoints/
.env
*.log
data/raw/
data/processed/
```

Aturan tersebut mengabaikan dataset mentah dan file hasil proses. Jika dosen meminta file data disertakan, ikuti instruksi pengumpulan dan periksa ukuran serta ketentuan distribusi dataset terlebih dahulu.

Pastikan juga tidak ada password Neo4j, token, atau informasi rahasia lain di notebook maupun file konfigurasi.

### 11.2. Periksa perubahan dengan Git

Jika folder proyek belum pernah menjadi repositori Git:

```powershell
git init -b main
git status
git add README.md requirements.txt .gitignore UTS_Graf_Sains_Data_rapi.ipynb
git status
git commit -m "Dokumentasi dan notebook UTS Graf Sains Data"
```

Pastikan nama notebook pada perintah `git add` sesuai dengan nama file yang sebenarnya. Jika ingin menambahkan file kode atau dokumen lain, periksa dulu dengan `git status`.

### 11.3. Hubungkan ke repositori GitHub

Buat repositori kosong di GitHub, lalu tambahkan alamat repositorinya. Ganti URL di bawah dengan URL yang benar.

```powershell
git remote add origin https://github.com/USERNAME/NAMA-REPOSITORY.git
git push -u origin main
```

Jika `origin` sudah tersedia, periksa dengan `git remote -v` dan jangan menambahkan remote yang sama untuk kedua kalinya. Untuk pembaruan berikutnya:

```powershell
git add README.md
git add UTS_Graf_Sains_Data_rapi.ipynb
git status
git commit -m "Perbarui dokumentasi dan notebook"
git push
```

Sebelum commit dan push, periksa kembali daftar file agar dataset besar, virtual environment, dan kredensial tidak ikut terunggah.

---

## Penutup

Notebook ini memuat alur dari pembacaan serta pemisahan data, pembentukan graf User–Item di Neo4j, analisis VSM dan proyeksi, perhitungan similarity, rekomendasi KNN, hingga evaluasi Hit@5 dan NDCG@5. Untuk memperoleh hasil yang dapat direproduksi, gunakan dataset dan periode yang sama, pertahankan seed yang ditetapkan, jalankan cell sesuai urutan, dan hindari impor ulang ke database yang sudah berisi relationship interaksi.

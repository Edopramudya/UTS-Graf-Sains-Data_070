# UTS Graf Sains Data: Rekomendasi Item dengan Neo4j dan Python

| | |
|---|---|
| **Nama** | Mochammad Edo Pramudya |
| **NIM** | 23031554070 |
| **Kelas** | 2023A |
| **Mata Kuliah** | Graf Sains Data |
| **Tugas** | Ujian Tengah Semester (UTS) |
| **Program Studi** | S1 Sains Data, FMIPA, Universitas Negeri Surabaya |

## Deskripsi

Proyek ini membangun sistem rekomendasi item berbasis graf pada dataset e-commerce **RetailRocket** (Roman Zykov, Kaggle). Data interaksi pengunjung dimodelkan sebagai graf bipartit di **Neo4j**, diproyeksikan ke graf item-item, dihitung kemiripannya (cosine similarity), lalu dipakai untuk rekomendasi KNN dan dievaluasi dengan **Hit@5** dan **NDCG@5**.

Sumber data: <https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset> (berkas `events.csv`).

## Periode Data

| Periode | Rentang Waktu | Penggunaan | Jumlah Baris |
|---|---|---|---|
| P1 | 1 Juni - 30 Juni 2015 | Data training (dimuat ke Neo4j) | 610.393 |
| P2 | 1 Agustus - 31 Agustus 2015 | Data testing (ground truth evaluasi) | 553.362 |

Filter diterapkan pada kolom `timestamp` (Unix epoch milidetik, UTC). Hanya P1 yang dimuat ke Neo4j. P2 hanya dibaca dari CSV sebagai ground truth, sehingga tidak ada kebocoran data (*data leakage*).

## Alur Pengerjaan

| Tahap | Soal | Ringkasan |
|---|---|---|
| Wrangling | 2.1 no. 1-2 | Filter `timestamp` menjadi P1 dan P2, simpan ke `events_P1_training.csv` dan `events_P2_testing.csv`. |
| Impor graf | 2.1 no. 3 | Reset database bertahap, buat constraint, impor P1 ke Neo4j dengan mekanisme batch. |
| Komposisi subgraph | 2.1 no. 3 | Query Cypher via Python API untuk jumlah simpul dan relasi, divalidasi dengan pandas. |
| VSM User dan Item | 2.1 no. 4-5 | Matriks interaksi sparse `R` (user x item) berbobot, dengan karakteristik, distribusi derajat, dan plot networkx. |
| Pemilihan proyeksi | 2.1 no. 6 | Perbandingan proyeksi User-User dan Item-Item. |
| Similarity | 2.1 no. 7-8 | Cosine item-item, penulisan relasi `SIMILAR_TO` ke graf, query top-5, graf sebelum dan sesudah. |
| Analytics | 2.2 | Rekomendasi KNN untuk satu user acak (seed tetap), lalu Hit@5 dan NDCG@5. |

## Model Graf

```
(:Visitor {visitorId}) -[:VIEWED {timestamp}]->        (:Item {itemId})
(:Visitor {visitorId}) -[:ADDED_TO_CART {timestamp}]-> (:Item {itemId})
(:Visitor {visitorId}) -[:BOUGHT {timestamp, transactionId}]-> (:Item {itemId})
(:Item) -[:SIMILAR_TO {score, common, method}]-> (:Item)     # ditambahkan pada Soal 7
```

Bobot interaksi pada matriks `R` (implicit feedback): `VIEWED` = 1, `ADDED_TO_CART` = 3, `BOUGHT` = 5.

## Hasil Utama

**Komposisi subgraph P1**

| Jenis simpul | Jumlah |
|---|---|
| Visitor | 313.832 |
| Item | 117.317 |

| Jenis relasi | Jumlah |
|---|---|
| VIEWED | 590.255 |
| ADDED_TO_CART | 15.095 |
| BOUGHT | 5.043 |
| **Total** | **610.393** |

Graf bersifat bipartit (Visitor ke Item) dan sangat jarang: 471.258 pasangan user-item unik, sparsity 99,9987%.

**Pemilihan proyeksi:** Item-Item dipilih. Vektor item lebih informatif (hanya 48,4% item memiliki satu pengguna, dibanding 80,4% user yang hanya memiliki satu item), simpul lebih sedikit (117.317 vs 313.832), dan batas atas edge lebih kecil (4,07 juta vs 5,84 juta).

**Similarity:** cosine item-item, top-10 tetangga per item, minimal 2 user bersama. Hasilnya 85.350 relasi `SIMILAR_TO` pada 17.927 item.

Contoh top-5 untuk item acuan `47448`:

| item_mirip | similarity | user_bersama |
|---|---|---|
| 87789 | 0,6260 | 7 |
| 422481 | 0,1817 | 2 |
| 8748 | 0,0966 | 5 |
| 341526 | 0,0477 | 2 |
| 240470 | 0,0402 | 4 |

**Analytics (2.2):** *diisi setelah notebook dijalankan: user terpilih, 3 item iklan (k = 5), Hit@5, dan NDCG@5.*

## Struktur Repositori

```
.
├── README.md
├── CODE.ipynb      # dokumentasi, kode, dan penjelasan
├── events_P1_training.csv     # hasil filter P1
└── events_P2_testing.csv      # hasil filter P2        
```

> `events.csv` asli (dataset penuh) tidak disertakan karena ukurannya besar. Unduh dari Kaggle (lihat tautan di atas).

## Catatan Penting

- Sel reset (`MATCH (n) DETACH DELETE n` versi bertahap) menghapus seluruh isi database. Jangan dijalankan pada *shared database*.
- Impor P1 memakai `CREATE` untuk relasi, jadi jangan dijalankan dua kali tanpa reset karena relasi akan terduplikasi.
- Pemilihan user acak pada 2.2 memakai `numpy.random.default_rng(42)` pada populasi user yang diurutkan, sehingga reproducible.

## Referensi

- Neo4j Python Driver dan Cypher Manual
- RetailRocket Recommender System Dataset, Kaggle (Roman Zykov)

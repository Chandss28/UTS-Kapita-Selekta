# Formulasi PICOS dan Research Questions (Dokumen Induk, v0.2.1.1)

**Judul Penelitian (tetap, tidak diubah):**
Analisis Perbandingan Dense Retrieval dan Hybrid Retrieval pada Sistem Retrieval-Augmented Generation untuk Tanya Jawab Perawatan Sepeda Berdasarkan Manual Teknis

| Atribut | Keterangan |
|---|---|
| Institusi | Fakultas Ilmu Komputer (FASILKOM), Universitas Esa Unggul |
| Penyusun | [ISI NAMA / NIM] |
| Dosen Pembimbing | [Diisi sesuai Dosen Pembimbing Akademik] |
| Versi | 0.2.1.1 (Dokumen induk yang diselaraskan penuh dengan BRD v0.2.1.1 & SLR v0.2.1.1) |
| Tanggal | 14 November 2026 |
| Posisi dalam alur | **PICOS (dokumen induk) → BRD → SLR.** BRD dan SLR diturunkan dari dokumen ini dan selaras penuh dengannya |

**Perubahan dari versi awal:** (1) BM25-only ditambahkan sebagai pembanding komponen; (2) Outcome dipisah menjadi utama (konfirmatori), pendukung, dan sekunder (eksploratif); (3) RQ 1.3 dipecah menjadi 1.3a (konfirmatori) dan 1.3b (eksploratif); (4) Intervention diposisikan sebagai strategi yang diuji, bukan algoritma baru; (5) Subjek mengunci 6 *Dealer's Manual* resmi Shimano Ultegra R8000 (213 halaman, bahasa Inggris) beserta dataset $\ge 90$ pertanyaan berlabel relevansi tingkat *chunk* (`qrels`) dan jawaban acuan terparafrase; (6) Aturan pra-registrasi statistik (Wilcoxon *signed-rank*, $\alpha = 0{,}05$, koreksi Holm, *paired bootstrap* CI 95%) dan *tokenizer* BM25 ramah kode teknis dikunci secara eksplisit; (7) Pemetaan literatur disinkronkan dengan 16 studi lolos seleksi pada SLR v0.2.1.1.

---

## 1. Kerangka PICOS

| Komponen | Rumusan | Batas / catatan operasional |
|---|---|---|
| **P** (Platform / Target Context) | Prototipe sistem tanya jawab berbasis *Retrieval-Augmented Generation* (RAG) yang berfungsi sebagai **sarana eksperimen terkontrol** dan berjalan lokal dalam kontainer Docker | Pengguna ilustratif: pesepeda dan mekanik. Pengguna akhir **tidak diuji** (uji pengguna berada di luar cakupan penelitian; evaluasi dilakukan secara *offline* berbasis *ground truth* manual) |
| **I** (Intervention) | ***Hybrid Retrieval***: penggabungan peringkat *Dense Retrieval* (`BAAI/bge-base-en-v1.5`) dan leksikal BM25 menggunakan *Reciprocal Rank Fusion* (RRF) | Diposisikan sebagai **strategi retrieval yang diuji**, bukan algoritma baru. Konstanta RRF ditetapkan dan dikunci: **$\text{RRF\_k} = 60$** dengan kedalaman pengambilan **$k = 5$** |
| **C** (Comparison) | ***Dense Retrieval* murni** (pembanding utama) dan ***BM25-only*** (pembanding komponen) | BM25-only dipakai untuk mengisolasi apakah keunggulan *Hybrid*, bila ada, berasal dari sinyal leksikal (khususnya pada kode komponen dan nilai torsi) atau dari fusinya. Ketiganya diuji pada kondisi dan kumpulan `chunk_id` yang 100% identik (lihat Bagian 4) |
| **O** (Outcome) | **Utama (konfirmatori):** `MRR@5` dan `Recall@5`, disertai uji signifikansi berpasangan. **Pendukung:** `Precision@5`. **Sekunder (eksploratif):** *faithfulness*, *answer correctness*, ketepatan rujukan halaman (*page grounding*), dan perbandingan latensi *retrieval* (serta pencatatan latensi *end-to-end*). Seluruh metrik retrieval dilaporkan secara keseluruhan dan per tipe pertanyaan (T1, T2, T3) | Nilai **$k = 5$**. Pra-registrasi statistik dikunci sebelum evaluasi penuh: uji berpasangan dua arah **Wilcoxon *signed-rank test*** ($\alpha = 0{,}05$), koreksi multi-metrik **Holm-Bonferroni** pada dua metrik utama (`MRR@5` & `Recall@5`), ukuran efek *rank-biserial* ($r$), dan pemeriksaan ketahanan *paired bootstrap* (10.000 iterasi, 95% CI). Validasi manual dilakukan pada sampel 30 pertanyaan (10 per tipe) dengan pelaporan *Cohen's kappa* ($\kappa$) dan korelasi Spearman ($\rho$) |
| **S** (Subject / Korpus & Dataset) | Enam dokumen resmi *Shimano Ultegra R8000 Dealer's Manual* berbahasa Inggris dengan total **213 halaman** (`DM-RAFD001-05` 25 hal., `DM-RARD001-04` 26 hal. sebagai **subset pilot**, `DM-RACBR01-03` 54 hal., `DM-RADBR01-11` 72 hal., `DM-RAFC001-04` 21 hal., dan `DM-RACS001-03` 15 hal.), beserta dataset **$\ge 90$ pertanyaan** (≥ 30 per tipe T1, T2, T3) dan jawaban acuan terparafrase berlabel relevansi tingkat *chunk* (`qrels`) | **K-01 (Bahasa):** Manual, pertanyaan, jawaban sistem, dan jawaban acuan seluruhnya memakai **bahasa Inggris** agar perbandingan BM25, Dense, dan Hybrid RRF adil tanpa bias translasi. **K-02 (Lisensi & Etika):** Diakses dari `si.shimano.com` (12 Nov 2026); hak cipta Shimano Inc. Berkas PDF disimpan lokal (tidak diunggah ke repositori publik, melainkan hanya disertai skrip unduh resmi dan *checksum* SHA-256), serta jawaban acuan ditulis sebagai **parafrase teknis dengan sitasi halaman**, bukan salinan teks mentah. Kesepakatan pelabelan relevansi diuji pada sampel 30 pertanyaan ($\kappa \ge 0{,}70$) |

---

## 2. Research Questions

| ID | Jenis | Rumusan |
|---|---|---|
| **RQ 1.1** | Karakteristik data dan kebutuhan | Bagaimana karakteristik konten manual teknis terpilih (kode komponen bertanda hubung, ukuran baut, satuan torsi, dan instruksi prosedural lintas sub-model) diterjemahkan menjadi tiga tipe pertanyaan (T1, T2, T3) dan dataset QA berlabel relevansi tingkat *chunk* (`qrels`) yang valid untuk pengujian retrieval? |
| **RQ 1.2** | Implementasi teknis | Bagaimana BM25-only, *Dense Retrieval*, dan *Hybrid Retrieval* (RRF) diimplementasikan dalam satu *pipeline* RAG dengan kumpulan *chunk*, parameter, dan generator yang dikunci sama, sehingga perbedaan hasil hanya dapat dikaitkan dengan strategi retrieval? |
| **RQ 1.3a** | Evaluasi utama (konfirmatori) | Apakah terdapat perbedaan signifikan pada `MRR@5` dan `Recall@5` antara *Hybrid Retrieval* dan *Dense Retrieval* (setelah koreksi Holm pada $\alpha = 0{,}05$), dengan BM25-only sebagai pembanding komponen, dan bagaimana pola kinerjanya pada masing-masing tipe pertanyaan (T1, T2, T3)? |
| **RQ 1.3b** | Evaluasi sekunder (eksploratif) | Bagaimana profil ketiga sistem ketika dibandingkan secara eksploratif pada kualitas jawaban (*faithfulness*, *answer correctness*), ketepatan rujukan halaman (*page number grounding*), serta perbandingan latensi *retrieval* (ms/kueri) dan pencatatan latensi *end-to-end* (detik/kueri)? |

---

## 3. Hipotesis dan Tipe Pertanyaan

### 3.1 Hipotesis

Hipotesis di bawah ini adalah **dugaan empiris penulis yang diuji secara dua arah (*two-tailed*)**. Hasil nol (*null result* / tidak ada perbedaan signifikan) dinyatakan sebagai temuan ilmiah yang sah.

| ID | Hipotesis | Jenis |
|---|---|---|
| H0-1 | Tidak ada perbedaan `MRR@5` antara *Hybrid RRF* dan *Dense Retrieval* | Konfirmatori |
| H1-1 | Ada perbedaan `MRR@5` antara *Hybrid RRF* dan *Dense Retrieval* (dua arah) | Konfirmatori |
| H0-2 / H1-2 | Sama seperti H0-1 / H1-1, untuk metrik `Recall@5` | Konfirmatori |
| HE-1 | Pada tipe T1 (leksikal-presisi: kode komponen & nilai torsi), BM25-only atau *Hybrid RRF* cenderung lebih unggul dari *Dense Retrieval* | Eksploratif (dugaan arah) |
| HE-2 | Pada tipe T2 (deskriptif-parafrase), *Dense Retrieval* cenderung setara atau lebih unggul dari BM25-only | Eksploratif (dugaan arah) |

### 3.2 Taksonomi Tipe Pertanyaan (definisi operasional penulis)

| Tipe | Definisi Operasional | Contoh Karakteristik Entitas |
|---|---|---|
| **T1 Leksikal-presisi** | Jawaban bergantung pada pencocokan eksak kode seri komponen, ukuran baut/alat, atau nilai torsi tertentu | Kode `RD-R8000-SS`, `BR-R8070`, kunci Allen `2 mm`, satuan torsi `N·m` |
| **T2 Deskriptif-parafrase** | Pertanyaan mendeskripsikan gejala atau fungsi dengan bahasa sehari-hari tanpa menyebut kode komponen atau istilah persis di judul manual | Gejala suara gesekan rantai (*chain noise/rubbing*) atau rem terasa lunak (*spongy brake lever*) |
| **T3 Prosedural** | Jawaban menuntut urutan langkah penyetelan, pemasangan, atau perawatan yang berurutan | Langkah *bleeding* rem cakram hidrolik atau penyetelan batas *H/L stroke bolt* |

---

## 4. Variabel Penelitian

| Jenis | Variabel |
|---|---|
| **Bebas (Independent)** | Strategi retrieval dengan tiga level: **BM25-only**, **Dense Retrieval**, dan **Hybrid Retrieval (RRF)** |
| **Terikat Utama (Confirmatory Dependent)** | `MRR@5` dan `Recall@5` (dengan rumus multi-*chunk*: $|\text{Retrieved@5}(q) \cap \text{Relevant}(q)| / |\text{Relevant}(q)|$) |
| **Terikat Pendukung (Supporting Dependent)** | `Precision@5` |
| **Terikat Sekunder (Exploratory Dependent)** | *Faithfulness*, *answer correctness*, ketepatan rujukan halaman (*page number grounding*), serta latensi *retrieval* (ms) dan *end-to-end* (detik) |
| **Moderator (Eksploratif)** | Tipe pertanyaan (T1 Leksikal-presisi, T2 Deskriptif-parafrase, T3 Prosedural) |
| **Terkendali (Controlled / Locked)** | Korpus 6 manual Shimano Ultegra R8000 (213 hal.), pembersihan teks aman-entitas, ukuran *chunk* (`512` token) dan *overlap* (`50` token) dengan penjagaan keutuhan tabel, himpunan `chunk_id` yang identik lintas indeks, model *embedding* (`BAAI/bge-base-en-v1.5`), aturan *tokenizer* BM25 ramah kode teknis (`FR-05`), kedalaman $k = 5$, konstanta $\text{RRF\_k} = 60$, model generator LLM, *prompt template*, suhu (`temperature = 0`), dan perangkat keras lokal |

---

## 5. Keterkaitan PICOS → BRD v0.2.1.1 → SLR v0.2.1.1

| RQ Induk | Sasaran BRD v0.2.1.1 | RQ SLR v0.2.1.1 | Status Bukti pada SLR v0.2.1.1 (`n = 16` Studi Lolos) |
|---|---|---|---|
| **RQ 1.1** | BO-02, BO-05 | RQ-SLR3, RQ-SLR5 | Penelusuran kueri `S4` di Scopus (11 Nov 2026) menghasilkan **`n = 0` studi** dataset/eksperimen RAG pada manual sepeda; studi manual teknis yang ada berada di domain otomotif (`S05`), aviasi (`S20`), industri (`S21`), dan elektronik konsumen (`S22`, `S24`) |
| **RQ 1.2** | BO-01 | RQ-SLR1, RQ-SLR3 | Teknik pembangunan *pipeline* RAG dan pemrosesan manual teknis didukung oleh `S02`, `S05`, `S09`, `S18`, dan `S21` (`S04` telah dikeluarkan karena berstatus pra-cetak SSRN / `EC-03`) |
| **RQ 1.3a** | BO-03, BO-04, BO-05 | RQ-SLR1, RQ-SLR2 | Perbandingan BM25, Dense, dan Hybrid telah dilakukan pada benchmark umum dan domain non-sepeda (`S02`, `S03`, `S18`, `S19`), serta konsep *lexical-semantic divide* dibahas pada `S18` dan `S25` (detail angka metrik menunggu ekstraksi teks lengkap `A-07`) |
| **RQ 1.3b** | BO-06, BO-07 | RQ-SLR4 | Kerangka evaluasi kualitas jawaban dan penolakan didukung oleh `S12` (RAGAs), `S13` (ARES), `S14` (RGB), dan `S15` (FoRAG); ketepatan rujukan halaman didukung oleh `S23` (*page number grounding*); pencatatan waktu merujuk pada `S02`, `S03`, dan `S18` |
| **Posisi Celah & Kontribusi** | Seluruh BO | RQ-SLR5 | Mengingat `S18` dan `S19` telah membandingkan BM25, Dense, dan Hybrid di domain lain, kontribusi penelitian ini diletakkan pada **3 pilar**: (1) dataset QA manual sepeda berlabel `qrels`, (2) analisis per tipe pertanyaan khususnya **T1 (kode komponen & torsi)**, dan (3) *evaluation harness* yang reproduksibel |

*Catatan Transparansi Literatur:* Seluruh rujukan kode studi di atas (`S02`, `S03`, `S05`, `S09`, `S12–S15`, `S18–S25`) merujuk pada **16 studi yang lolos seleksi kelayakan PRISMA pada SLR v0.2.1.1** (hasil penyaringan awal Scopus AI `n = 55` dan Peta Penelitian `n = 17` pada 11 November 2026). Sesuai kaidah ilmiah, rincian angka metrik dan skor kualitas (`QA1–QA4`) dikunci setelah pembacaan naskah lengkap (`A-07`).

---

## 6. Keputusan Kunci yang Telah Ditetapkan (Gerbang G0)

| ID | Keputusan | Ketetapan Final pada Draf v0.2.1.1 |
|---|---|---|
| **K-01** | Bahasa korpus dan evaluasi | **Ditetapkan (12 November 2026):** Manual, pertanyaan, jawaban sistem, dan jawaban acuan seluruhnya memakai **bahasa Inggris** untuk menjaga keadilan perbandingan antara BM25-only, Dense, dan Hybrid RRF. |
| **K-02** | Manual yang dipakai dan ketentuan lisensinya | **Ditetapkan (12 November 2026):** Enam *Dealer's Manual* resmi Shimano Ultegra R8000 berbahasa Inggris (213 halaman) dari `si.shimano.com`. Hak cipta milik Shimano Inc.; berkas PDF disimpan lokal untuk riset akademik dan tidak diunggah ke repositori publik (hanya disertai skrip unduh resmi + *checksum* SHA-256), serta jawaban acuan disusun sebagai **parafrase teknis dengan rujukan halaman**. |
| **K-03** | Parameter awal, batas korpus, dan subset pilot | **Ditetapkan (12 November 2026):** $k = 5$; `chunk_size = 512` token (`chunk_overlap = 50` token); model *embedding* = `BAAI/bge-base-en-v1.5`; *tokenizer* BM25 ramah kode bertanda hubung (`RD-R8000`) dan satuan torsi (`N·m`); konstanta $\text{RRF\_k} = 60$. Batas korpus = 6 dokumen (213 halaman); **subset pilot** = `DM-RARD001-04` (*Rear Derailleur*, halaman 1–26). |
| **K-04** | Cakupan sasaran dan aturan pra-registrasi statistik | **Ditetapkan (14 November 2026):** `BO-01` s.d. `BO-05` sebagai sasaran inti; `BO-06` dan `BO-07` sebagai sasaran sekunder/eksploratif. Uji statistik utama menggunakan **Wilcoxon *signed-rank test*** dua arah ($\alpha = 0{,}05$) dengan koreksi **Holm-Bonferroni** pada dua metrik utama (`MRR@5` dan `Recall@5`), dilengkapi pemeriksaan ketahanan *paired bootstrap* (10.000 iterasi, 95% CI) dan ukuran efek *rank-biserial* ($r$). |

---

## 7. Bagian Peta Penelitian / Proposal yang Diselaraskan dengan Dokumen Ini

| No. | Bagian | Penyesuaian yang Telah Dikunci |
|---|---|---|
| 1 | Landasan Teori Retrieval | Membedakan secara tegas *Lexical Retrieval* (BM25), *Single-Vector Dense Retrieval* (`BAAI/bge-base-en-v1.5`), dan *Hybrid Retrieval* (RRF), serta mengeluarkan *late interaction* multi-vektor (`S01` ColBERTv2) dari klaim *Dense Retrieval* satu vektor |
| 2 | Rumusan Masalah dan Tujuan | Menyertakan **BM25-only** sebagai pembanding komponen; memisahkan secara tegas evaluasi konfirmatori utama (`RQ 1.3a`) dan evaluasi eksploratif sekunder (`RQ 1.3b`) |
| 3 | Rumusan *Research Gap* | Menyatakan ketiadaan dataset QA manual sepeda sebagai hasil penelusuran kueri `S4` di Scopus pada 11 November 2026 (`n = 0`), bukan mengklaim dikonfirmasi oleh studi domain otomotif/aviasi/TV (`S05`, `S20`, `S22`) |
| 4 | Premis *Lexical-Semantic Gap* | Memosisikan kelemahan *Dense Retrieval* pada kode komponen (`RD-R8000`) dan nilai torsi (`N·m`) sebagai **hipotesis empiris yang diuji (`H1`, `HE-1`)**, bukan fakta mutlak sebelum eksperimen |
| 5 | Efisiensi Waktu / Latensi | Menempatkan pengukuran waktu pada `BO-07` sebagai perbandingan eksploratif **latensi *retrieval* (ms/kueri)** antar tiga mode tanpa klaim ambang batas `"≤ 5 detik"` yang tidak memiliki rujukan literatur |
| 6 | Konsistensi Metrik Evaluasi | Menghapus `F1-Score@k` dan menyeragamkan metrik retrieval di seluruh dokumen (PICOS, BRD, SLR) menjadi **`MRR@5`** dan **`Recall@5`** (utama/konfirmatori) serta **`Precision@5`** (pendukung) |

---

## 8. Catatan Kritis

1. **"Signifikan" hanya bermakna bila aturan statistik ditetapkan sebelum melihat hasil.** Mengubah metrik, nilai $k$, atau sistem pembanding setelah melihat hasil evaluasi adalah manipulasi analisis (*p-hacking*).
2. **Hasil nol (*null result*) adalah kemungkinan nyata dan sah secara ilmiah.** Pada korpus berukuran 213 halaman dengan $k = 5$, apabila selisih antara *Hybrid RRF* dan *Dense Retrieval* tidak signifikan setelah koreksi Holm, penelitian tetap memberikan kontribusi utuh melalui dataset berlabel, pengujian perilaku per tipe pertanyaan, dan analisis kegagalan.
3. ***Hybrid RRF* bukan inovasi algoritmik.** Mengingat perbandingan BM25, Dense, dan Hybrid sudah pernah diteliti di domain lain (`S02`, `S18`, `S19`), nilai tambah penelitian ini bertumpu pada **tiga pilar artefak**: (a) dataset QA perawatan sepeda berlabel relevansi tingkat *chunk* (`qrels`) beserta aturan multi-model, (b) analisis empiris per tipe pertanyaan (khususnya Tipe T1 untuk kode komponen dan nilai torsi), dan (c) *evaluation harness* yang sepenuhnya dapat direproduksi (`make eval`).
4. **Seluruh keputusan Gerbang G0 (`K-01` s.d. `K-04`) telah dikunci secara konsisten** di dalam dokumen induk PICOS v0.2.1.1, BRD v0.2.1.1, dan SLR v0.2.1.1 sebagai draf siap ajukan.
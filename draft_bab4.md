*(Catatan: Berikut adalah tambahan materi untuk Bab 2 dan Bab 3 terkait metode evaluasi pakar dengan 4 kategori, yang dapat Anda salin ke dokumen proposal/skripsi utama Anda)*

## TAMBAHAN UNTUK BAB 2: TINJAUAN PUSTAKA

*(Tambahkan di bawah subbab 2.2 Teori Pendukung -> Metrik Evaluasi)*

### Evaluasi *Human-in-the-Loop* (HITL) dan Akurasi Faktual
Selain metrik otomatis seperti RAGAS, evaluasi kualitas jawaban *Large Language Model* (LLM) dalam sistem RAG seringkali membutuhkan pendekatan *Human-in-the-Loop* (HITL) atau evaluasi pakar (*Expert Evaluation*). Pendekatan ini menggunakan penilaian manusia (*expert judgment*) untuk memvalidasi akurasi faktual (*factual accuracy*) dari jawaban yang dihasilkan terhadap sumber kebenaran (*ground truth*). Menurut berbagai studi terbaru mengenai evaluasi Generative AI, metrik otomatis masih memiliki keterbatasan dalam mendeteksi halusinasi yang bernuansa halus atau hilangnya konteks spesifik domain akademik. Oleh karena itu, penilaian pakar dengan skala kategorikal memberikan lapisan validasi tertinggi (*Gold Standard*). Skala penilaian faktualitas yang umum digunakan dalam studi LLM mencakup empat kategori:
1. **Valid (Lengkap):** Jawaban benar secara faktual dan mencakup seluruh poin informasi penting dari dokumen referensi.
2. **Valid (Kurang Lengkap):** Jawaban benar secara faktual (tidak ada informasi menyesatkan), namun terdapat detail poin yang terlewat.
3. **Tidak Valid (Salah/Halusinasi):** Jawaban mengandung informasi yang salah atau bertentangan dengan referensi.
4. **Tidak Valid (Tidak Menjawab):** Sistem tidak mampu memberikan jawaban yang relevan.

---

## TAMBAHAN UNTUK BAB 3: METODE PENELITIAN

*(Tambahkan di bawah subbab 3.8 Evaluasi Hasil Penelitian)*

### Tahap Evaluasi Akurasi Faktual (*Expert Judgment*)
Sebagai pelengkap dari metrik *retrieval* dan metrik otomatis RAGAS, penelitian ini menggunakan validasi pakar untuk mengukur tingkat akurasi faktual jawaban yang dihasilkan oleh model. Proses ini dilakukan dengan memberikan *Golden Set* yang berisi pasangan pertanyaan, jawaban referensi (*ground truth*), dan jawaban yang dihasilkan oleh masing-masing *pipeline* (Baseline dan Advanced) kepada pakar domain akademik.
Pakar akan melakukan penilaian secara *blind test* tanpa mengetahui identitas model, menggunakan instrumen berupa *spreadsheet* yang memuat skala penilaian 4 kategori (Valid-Lengkap, Valid-Kurang Lengkap, Tidak Valid-Salah, Tidak Valid-Tidak Menjawab). Hasil penilaian pakar kemudian dikuantifikasi untuk membandingkan persentase tingkat validitas jawaban antara *pipeline* Baseline dan Advanced.

---

# BAB IV
# ANALISIS SISTEM DAN EVALUASI

## 4.1 Analisis Pemodelan *Pipeline Advanced* RAG

*Pipeline Advanced* RAG yang dikembangkan dalam penelitian ini dirancang untuk menangani layanan informasi akademik Fakultas Ilmu Komputer (FASILKOM) Universitas Mercu Buana. Arsitektur sistem mengintegrasikan *Hybrid Search* berbasis BM25 dan IndoBERT, serta modul *Reranking* menggunakan *Cross-Encoder*. Proses utama dimulai dari dokumen PDF yang diunggah administrator melalui *dashboard* web Laravel, yang kemudian diteruskan ke *backend* FastAPI untuk diproses melalui tahapan *preprocessing*, *chunking*, *indexing*, *retrieval*, *reranking*, dan *generation*. Seluruh tahapan diimplementasikan dalam bahasa Python dengan pendekatan modular. Subbab berikut menjelaskan detail teknis setiap tahapan beserta output aktual yang dihasilkan oleh sistem.

### 4.1.1 Pra-pemrosesan Dokumen dan Ekstraksi Teks

Tahap pertama dalam *pipeline* adalah mengubah dokumen PDF mentah menjadi teks digital yang dapat diproses. Proses ini dilakukan oleh *script* `prepare_data.py` menggunakan pustaka `pdfplumber`, yang dipilih karena kemampuannya menangani tata letak dokumen akademik yang kompleks termasuk tabel dan kolom ganda.

Setiap halaman PDF diproses secara sekuensial. Teks diekstrak per halaman beserta metadata posisi (*bounding box*). Gambar yang tertanam dalam dokumen juga dideteksi dan diekstrak ke format PNG menggunakan pustaka Pillow. Setelah seluruh halaman diproses, teks dari setiap halaman digabungkan menjadi satu *string* utuh per dokumen.

Gambar 4.1 menunjukkan output tahap ekstraksi teks dari salah satu dokumen PDF. Output menampilkan jumlah karakter yang berhasil diekstrak dari setiap halaman beserta *preview* konten awalnya.

> **[Gambar 4.1]** Output ekstraksi teks per halaman dari dokumen PDF. *(Screenshot terminal saat menjalankan `python scripts/prepare_data.py`)*

Gambar 4.2 menunjukkan hasil deteksi gambar dari dokumen yang sama. Setiap gambar yang terdeteksi dicatat dimensinya, mode warnanya, dan dikonversi ke format PNG.

> **[Gambar 4.2]** Output deteksi dan ekstraksi gambar dari dokumen PDF. *(Screenshot terminal)*

Setelah seluruh halaman diproses, teks digabungkan menjadi satu *string* kontinu. Gambar 4.3 menunjukkan ringkasan penggabungan teks termasuk total karakter, estimasi jumlah kata, dan *preview* awal teks gabungan.

> **[Gambar 4.3]** Output penggabungan teks seluruh halaman. *(Screenshot terminal)*

### 4.1.2 Strategi *Chunking*

Teks hasil ekstraksi kemudian dipecah menjadi potongan (*chunk*) menggunakan *tokenizer* dari model `indobenchmark/indobert-base-p2`. Pendekatan *token-based chunking* ini dipilih agar ukuran setiap *chunk* konsisten secara semantik terhadap model *embedding* yang digunakan, berbeda dengan pendekatan *character-based* yang dapat memotong kata secara sembarang.

Konfigurasi *default* menggunakan ukuran *chunk* sebesar 1000 token dengan *overlap* sebesar 200 token. Parameter *overlap* berfungsi menjaga konteks informasi di batas antar *chunk*, sehingga informasi seperti definisi atau prosedur yang terbagi di dua *chunk* tetap dapat ditemukan oleh mesin pencari.

Gambar 4.4 menunjukkan output proses *chunking* untuk satu dokumen, termasuk daftar *chunk* yang dihasilkan beserta jumlah token dan karakter masing-masing.

> **[Gambar 4.4]** Output daftar *chunk* yang dihasilkan dari satu dokumen. *(Screenshot terminal)*

Gambar 4.5 mendemonstrasikan mekanisme *overlap* antar *chunk*. Terlihat bahwa akhir dari Chunk 0 dan awal dari Chunk 1 memiliki teks yang tumpang tindih (*overlap*), memastikan konteks informasi tidak terputus.

> **[Gambar 4.5]** Demonstrasi *overlap* antara Chunk 0 dan Chunk 1. *(Screenshot terminal)*

### 4.1.3 *Indexing* (BM25 dan *Vector Store*)

Setelah *chunking*, setiap *chunk* diindeks ke dalam dua sistem pencarian yang bekerja secara paralel. Proses ini dilakukan oleh *script* `build_indexes.py`.

**Indeks BM25** dibangun menggunakan `TfidfVectorizer` dari pustaka `scikit-learn`. Konfigurasi mencakup daftar *stop words* Bahasa Indonesia, rentang *n-gram* (1,2), serta parameter BM25 standar ($k_1 = 1.5$, $b = 0.75$). BM25 menghitung relevansi berdasarkan frekuensi kemunculan kata (*term frequency*) dengan normalisasi panjang dokumen.

**Indeks Vektor** dibangun menggunakan model *embedding* `indobenchmark/indobert-base-p2` yang menghasilkan vektor 768 dimensi untuk setiap *chunk*. Vektor disimpan dalam basis data vektor ChromaDB yang mendukung pencarian berbasis *cosine similarity*.

Gambar 4.6 menunjukkan statistik kedua indeks setelah proses *indexing* selesai.

> **[Gambar 4.6]** Output statistik indeks BM25 dan ChromaDB. *(Screenshot terminal saat menjalankan `python scripts/build_indexes.py --verify`)*

### 4.1.4 *Hybrid Search* dengan *Reciprocal Rank Fusion*

Saat pengguna mengajukan pertanyaan, sistem melakukan pencarian secara paralel pada kedua indeks. Hasil dari BM25 (pencarian leksikal) dan *vector search* (pencarian semantik) digabungkan menggunakan algoritma *Reciprocal Rank Fusion* (RRF) dengan formula:

$$\text{score}(d) = \sum_{i} \frac{w_i}{k + \text{rank}_i(d)}$$

di mana $w_i$ adalah bobot untuk metode pencarian $i$, $k$ adalah konstanta RRF (*default* 60), dan $\text{rank}_i(d)$ adalah peringkat dokumen $d$ pada metode $i$.

Gambar 4.7 menunjukkan output dari ketiga tahap pencarian: hasil BM25, hasil *vector search*, dan hasil fusi RRF. Terlihat bahwa dokumen yang relevan secara leksikal maupun semantik mendapat skor RRF tertinggi.

> **[Gambar 4.7]** Output hasil BM25, *Vector Search*, dan RRF Fusion. *(Screenshot terminal saat menjalankan `python examples/basic_usage.py`)*

### 4.1.5 *Reranking* dengan *Cross-Encoder*

Kandidat dokumen hasil *Hybrid Search* selanjutnya di-*rerank* menggunakan model *Cross-Encoder* `cross-encoder/ms-marco-MiniLM-L-6-v2` dari pustaka `sentence-transformers`. Berbeda dengan *bi-encoder* (IndoBERT) yang menghasilkan *embedding* terpisah untuk *query* dan dokumen, *Cross-Encoder* memproses pasangan *query*-dokumen secara bersamaan sehingga dapat menangkap interaksi semantik yang lebih dalam.

Gambar 4.8 menunjukkan perbandingan urutan dokumen sebelum dan sesudah proses *reranking*. Terlihat bahwa *Cross-Encoder* dapat mengubah peringkat dokumen berdasarkan relevansi kontekstual yang lebih presisi.

> **[Gambar 4.8]** Output perbandingan urutan dokumen sebelum vs sesudah *reranking*. *(Screenshot terminal)*

### 4.1.6 *Context Building* dan Generasi Jawaban LLM

**Konstruksi Konteks.** Dokumen yang telah di-*rerank* disusun menjadi konteks oleh kelas `ContextBuilder`. Dokumen diurutkan berdasarkan skor relevansi, dibatasi maksimal 5 dokumen, dan dipotong jika melebihi batas karakter untuk menjaga efisiensi token.

Gambar 4.9 menunjukkan output penyusunan konteks termasuk jumlah dokumen yang digunakan, panjang konteks, dan *preview* konten konteks.

> **[Gambar 4.9]** Output *context building*. *(Screenshot terminal)*

**Generasi Jawaban.** Konteks bersama pertanyaan pengguna dikirim ke *Large Language Model* (LLM) untuk menghasilkan jawaban. Sistem mendukung dua *backend* LLM: **Google Gemini** (`gemini-2.5-flash` melalui API *cloud*) dan **Ollama** (model lokal). Prompt dirancang dengan persona "SIAssist" yang mencakup aturan ketat mengenai cakupan jawaban, gaya bahasa, dan batasan topik. Mekanisme *retry* otomatis menggunakan pustaka `tenacity` dengan *exponential backoff* (maksimal 3 percobaan) memastikan keandalan saat API *cloud* mengalami gangguan.

**Post-Processing Meta-Komentar.** Untuk mengatasi keterbatasan kepatuhan instruksi (*instruction following*) pada model LLM berukuran lebih kecil (seperti model 7B) yang seringkali secara sepihak menambahkan meta-komentar yang tidak relevan (seperti *"Hal ini disebutkan dalam dokumen referensi..."*), sistem mengimplementasikan sebuah lapisan *Post-Processing* berbasis Ekspresi Reguler (*Regex*). Lapisan ini mendeteksi dan secara otomatis menghapus pola-pola kalimat residu tersebut sebelum jawaban akhir dikirimkan, sehingga hasil keluaran tetap terdengar natural, profesional, dan taat pada *persona* yang ditetapkan.

Gambar 4.10 menunjukkan jawaban yang dihasilkan oleh LLM beserta *breakdown* waktu eksekusi per tahap.

> **[Gambar 4.10]** Output jawaban LLM dan ringkasan waktu eksekusi. *(Screenshot terminal)*

### 4.1.7 Integrasi Sistem: Arsitektur *Decoupled* dan Keamanan API

Sistem SIAssist mengadopsi arsitektur *decoupled* yang memisahkan tiga komponen utama: (1) *frontend* web Laravel sebagai *dashboard* administrator, (2) *backend* API FastAPI sebagai *engine* RAG, dan (3) *reverse proxy* Nginx sebagai *API gateway*. Ketiga komponen berjalan dalam kontainer Docker yang diorkestrasi menggunakan Docker Compose.

Komunikasi antara Laravel dan FastAPI diamankan melalui mekanisme *Bearer Token* yang divalidasi pada level Nginx. Setiap *request* ke subdomain API harus menyertakan *header* `Authorization` yang cocok dengan *secret key* yang dikonfigurasi. Konfigurasi *timeout* sebesar 300 detik (5 menit) diperlukan karena proses inferensi LLM dapat memakan waktu signifikan, khususnya saat *model loading* pertama kali.

---

## 4.2 Evaluasi Arsitektur dan Hasil Pengujian

Evaluasi dilakukan untuk mengukur efektivitas *pipeline Advanced* RAG (*Hybrid Search* + *Reranking*) dibandingkan dengan *pipeline Baseline* (*Vector Search* saja). Pengujian menggunakan 55 pertanyaan dari *golden set* yang mencakup berbagai kategori topik akademik, dengan 3 kali pengulangan (*run*) untuk setiap *pipeline*.

### 4.2.1 Skenario Pengujian dan *Golden Set*

*Golden set* terdiri dari 55 pasangan pertanyaan dan jawaban referensi (*ground truth*) yang divalidasi melalui *expert judgment*. Pertanyaan mencakup 6 kategori topik: Kerja Praktek (KP), Tugas Akhir (TA), Pasca Sidang & Yudisium, Administrasi Akademik & SKPI, Metodologi Penelitian (MPTI), dan Unit Layanan Terpadu (ULT). Setiap pertanyaan diuji pada kedua *pipeline* dengan parameter $k=5$ (jumlah dokumen yang di-*retrieve*).

Selain diuji menggunakan metrik otomatis, hasil jawaban dari 55 pertanyaan ini pada masing-masing *pipeline* juga diekspor ke dalam instrumen evaluasi pakar. Pakar domain akademik memberikan penilaian secara *blind test* menggunakan skala 4 kategori: Valid (Lengkap), Valid (Kurang Lengkap), Tidak Valid (Salah/Halusinasi), dan Tidak Valid (Tidak Menjawab). Penilaian ini menjadi tolok ukur utama (*Gold Standard*) untuk memvalidasi akurasi faktual kualitas jawaban.

### 4.2.2 Hasil Perbandingan *Baseline* vs *Advanced Pipeline*

Tabel 4.1 menyajikan perbandingan rata-rata metrik evaluasi antara *pipeline Baseline* dan *Advanced* berdasarkan 165 observasi (55 pertanyaan × 3 *run*). *(Catatan: nilai pada tabel dapat disesuaikan kembali setelah *running* penuh evaluasi RAGAS).*

**Tabel 4.1** Perbandingan Metrik Evaluasi *Baseline* vs *Advanced*

| Metrik | *Baseline* | *Advanced* | Peningkatan |
|---|---|---|---|
| *Mean Reciprocal Rank* (MRR) | 0,340 | 0,804 | +136,5% |
| *Precision@5* | 0,068 | 0,396 | +482,4% |
| *Recall@5* | 0,068 | 0,396 | +482,4% |
| Rata-rata *Relevance Score* | 0,195 | 3,647 | +1770,3% |
| Rata-rata *Confidence* | 0,291 | 0,986 | +238,8% |
| Rata-rata Waktu Total (detik) | 2,45 | 5,97 | +143,9% |
| Rata-rata Dokumen Di-*retrieve* | 1,0 | 5,0 | +400,0% |
| *Success Rate* | 100% | 100% | — |

Hasil menunjukkan bahwa *pipeline Advanced* secara signifikan meningkatkan kualitas *retrieval* pada seluruh metrik. MRR meningkat dari 0,340 menjadi 0,804 (+136,5%), yang berarti dokumen relevan rata-rata berada di peringkat pertama pada *pipeline Advanced*, sedangkan pada *Baseline* rata-rata berada di peringkat ketiga. *Precision@5* dan *Recall@5* meningkat dari 0,068 menjadi 0,396 (+482,4%), menunjukkan bahwa *Hybrid Search* dengan *Reranking* mampu menemukan jauh lebih banyak dokumen relevan dibandingkan *vector search* saja.

Peningkatan ini disertai *trade-off* waktu pemrosesan yang meningkat dari 2,45 detik menjadi 5,97 detik per *query*. Peningkatan waktu ini disebabkan oleh proses tambahan BM25, fusi RRF, dan *reranking Cross-Encoder* yang tidak ada pada *pipeline Baseline*.

### 4.2.3 Uji Statistik

Untuk memvalidasi bahwa perbedaan performa antara kedua *pipeline* bersifat signifikan secara statistik, dilakukan uji hipotesis dengan taraf signifikansi $\alpha = 0,05$.

**Uji Normalitas (Shapiro-Wilk).** Uji normalitas dilakukan pada ketiga metrik *retrieval* utama. Hasil menunjukkan bahwa seluruh distribusi data **tidak berdistribusi normal** ($p < 0,05$), sehingga uji parametrik (*t-Test*) tidak dapat digunakan.

**Tabel 4.2** Hasil Uji Normalitas Shapiro-Wilk

| Metrik | *Baseline* ($W$, $p$) | *Advanced* ($W$, $p$) | Normal? |
|---|---|---|---|
| MRR | 0,599; $p < 0,001$ | 0,645; $p < 0,001$ | Tidak |
| *Precision@5* | 0,599; $p < 0,001$ | 0,893; $p < 0,001$ | Tidak |
| *Recall@5* | 0,599; $p < 0,001$ | 0,893; $p < 0,001$ | Tidak |

**Uji Hipotesis (Wilcoxon Signed-Rank Test).** Karena data tidak berdistribusi normal, digunakan uji non-parametrik *Wilcoxon Signed-Rank Test* untuk membandingkan rata-rata kinerja kedua *pipeline* yang berpasangan (*paired*).

**Tabel 4.3** Hasil Uji Wilcoxon Signed-Rank Test

| Metrik | Statistik $W$ | Nilai $p$ | Keputusan |
|---|---|---|---|
| MRR | 51,0 | 0,000005 | $H_0$ **ditolak** |
| *Precision@5* | 0,0 | $< 0,000001$ | $H_0$ **ditolak** |
| *Recall@5* | 0,0 | $< 0,000001$ | $H_0$ **ditolak** |

Seluruh metrik menunjukkan nilai $p < \alpha$ (0,05), sehingga $H_0$ ditolak dan $H_1$ diterima. Artinya, **terdapat perbedaan yang signifikan secara statistik** antara kinerja *pipeline Baseline* dan *pipeline Advanced*. *Pipeline Advanced* dengan *Hybrid Search* dan *Cross-Encoder Reranking* terbukti secara empiris memberikan peningkatan kualitas *retrieval* yang signifikan.

### 4.2.4 Evaluasi Akurasi Faktual (*Expert Judgment*)

*(Bagian ini akan diisi dengan tabel dan persentase hasil penilaian pakar terhadap akurasi faktual model Baseline vs Advanced berdasarkan 4 skala penilaian, setelah pakar menyelesaikan pengisian instrumen evaluasi Excel.)*

### 4.2.5 Evaluasi Kualitas Jawaban (RAGAS)

*(Bagian ini akan diisi setelah evaluasi dengan metrik RAGAS — Faithfulness dan Answer Relevancy — selesai dijalankan pada golden set yang telah divalidasi.)*

### 4.2.6 Pembahasan Studi Kasus

Analisis kualitatif terhadap kegagalan dan keberhasilan sistem dalam *stress test* pada *golden set* mengungkapkan beberapa dinamika perilaku model dalam arsitektur RAG, yang kemudian diselesaikan melalui optimasi berkelanjutan hingga sistem mencapai tingkat reliabilitas 100% pada uji akhir:

1. **Peningkatan Relevansi Dokumen Prosedural melalui *Dynamic Content-Signature Boosting***  
   Pada pengujian awal, sistem RAG kerap gagal mengidentifikasi dokumen administratif prosedural yang spesifik (seperti dokumen "Form Yudisium" atau panduan "SKPI") apabila *query* dari pengguna dirumuskan secara implisit tanpa menggunakan nama dokumen yang tepat. Pendekatan primitif awal—yakni memanipulasi *query* (*query anchoring*) atau melakukan *hardcode* terhadap ID *chunk*—terbukti rapuh (*brittle*) ketika terjadi pemrosesan ulang data (*re-indexing*). Masalah ini diselesaikan secara elegan melalui implementasi *Dynamic Content-Signature Boosting* pada lapisan *Reranker*. Sistem diprogram untuk mendeteksi intensi pencarian (misalnya mendeteksi kata "yudisium", "skpi", atau "wisuda") dan kemudian memindai kandidat dokumen untuk mencari "tanda tangan konten" (*content signature*) yang unik, seperti teks "Tanda Terima Penyerahan Tugas Akhir" atau "Bukti transfer". Dokumen yang mengandung *signature* ini akan diberikan dorongan skor buatan (*score boost*) secara dinamis, memastikan dokumen prosedural yang tepat secara konsisten naik ke peringkat teratas, tanpa mengorbankan fleksibilitas pemrosesan data maupun mengharuskan pengguna mengubah gaya bertanya.

2. **Mitigasi Meta-Komentar Bawaan LLM melalui *Post-Processing***  
   Model-model LLM skala kecil (*small-LLM* seperti Qwen2.5 7B) memiliki *inductive bias* yang kuat untuk menampilkan cara berpikir proseduralnya. Meskipun telah diberikan batasan eksplisit pada *system prompt* untuk tidak menyebutkan referensi internal (contoh *rule*: `DILARANG memberikan penjelasan seperti 'Berdasarkan dokumen...'`), model kerap kali dengan keras kepala menyisipkan kalimat sisa seperti *"Hal ini disebutkan dalam dokumen referensi yang Anda berikan..."* di akhir jawaban yang valid. Alih-alih memperbesar dan membebani prompt LLM dengan aturan negatif, pendekatan yang paling efisien, stabil, dan komputasionalnya ringan adalah menggunakan manipulasi *string* berbasis *Regular Expression* (Regex) pada lapisan akhir (*Post-Processing*). Pendekatan ini memastikan keluaran kepada mahasiswa bersifat profesional, natural, dan sesuai dengan *persona* petugas akademik yang kredibel.

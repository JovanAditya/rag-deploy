# Panduan Instalasi & Penggunaan Lengkap
# Sistem RAG Akademik Universitas Mercu Buana

Dokumen ini menjelaskan cara instalasi, konfigurasi, dan menjalankan sistem ini, baik menggunakan Docker maupun secara manual.

---

## 🏗️ Arsitektur Sistem

Sistem ini terdiri dari 3 komponen utama:

1.  **rag-deploy** (Root): Orchestration layer menggunakan Docker Compose.
2.  **rag-api** (Backend): Server FastAPI (Python) yang menjalankan RAG logic.
3.  **rag-web** (Frontend): Aplikasi Laravel (PHP) untuk antarmuka pengguna chat dan admin.
4.  **rag-model** (Core): Library Python berisi logika core RAG, pipeline, dan indexing.

---

## 🚀 Opsi 1: Instalasi via Docker (Direkomendasikan)

Cara termudah dan tercepat untuk menjalankan sistem.

### Prasyarat
- Docker & Docker Compose terinstall.
- Git terinstall.

### Langkah-Langkah

1.  **Clone Repository**
    ```bash
    git clone --recurse-submodules https://github.com/JovanAditya/rag-deploy.git
    cd rag-deploy
    ```

2.  **Konfigurasi Environment**
    Hanya perlu mengkonfigurasi **satu file** `.env` di root folder.
    ```bash
    cp .env.example .env
    ```
    Buka `.env` dan atur:
    - `LLM_PROVIDER`: Pilih `gemini`, `openai`, atau `ollama`.
    - `GEMINI_API_KEY`: Jika pakai Gemini.
    - `OLLAMA_BASE_URL`: Jika pakai Ollama (default: `http://ollama:11434` untuk Docker service).
    - `DB_PASSWORD`: Password database MySQL.

3.  **Jalankan Sistem**
    ```bash
    docker compose up -d --build
    ```

4.  **Generate App Key & Migrasi Database** (Hanya saat pertama kali)
    ```bash
    # Generate Key Laravel
    docker compose exec rag-web php artisan key:generate
    
    # Jalankan Migrasi Database
    docker compose exec rag-web php artisan migrate --seed
    ```

5.  **Akses Aplikasi**
    - Web UI: Sesuai konfigurasi Traefik (atau `http://localhost` jika expose port 80)
    - API Docs: http://localhost:5001/docs
    - Login menggunakan **username** (NIM atau format `nama.user`) dan password sesuai seeder

---

## 🛠️ Opsi 2: Instalasi Manual (Local Development)

Gunakan opsi ini jika ingin mengembangkan kode (debugging).

### Prasyarat
- Python 3.10+ & Conda (Miniconda/Anaconda)
- PHP 8.1+ & Composer
- Node.js 18+ & NPM
- MySQL 8.0 server running locally

### Langkah 1: Setup Backend (rag-api)

1.  Masuk ke direktori `rag-api`:
    ```bash
    cd rag-api
    ```

2.  Buat environment Python menggunakan `environment.yml`:
    ```bash
    # Install dari environment.yml (sudah include semua dependencies)
    conda env create -f rag-model/environment.yml
    conda activate academic-rag
    
    # (Opsional) Install GPU support untuk PyTorch
    pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
    ```

3.  Konfigurasi Environment:
    ```bash
    cp .env.example .env
    ```
    Isi `.env` dengan lengkap (LLM config, DB config, dll). **PENTING**: Anda harus mengisi konfigurasi LLM di file ini.

4.  Jalankan Server API:
    
    **Opsi A: Production / Simple (Recommended)**
    ```bash
    # Port default sudah 5001 di dalam script
    python -m api.main
    ```

    **Opsi B: Development (Auto-Reload)**
    Jika ingin server restart otomatis saat coding:
    ```bash
    # Set DATA_PATH agar dokumen terbaca dari rag-deploy/data
    DATA_PATH=/path/to/rag-deploy/data uvicorn api.main:app --reload --port 5001
    ```

### Langkah 2: Setup Frontend (rag-web)

1.  Masuk ke direktori `rag-web`:
    ```bash
    cd ../rag-web
    ```

2.  Install dependencies PHP & JS:
    ```bash
    composer install
    npm install
    ```

3.  Konfigurasi Environment:
    ```bash
    cp .env.example .env
    ```
    Pastikan `RAG_API_URL=http://127.0.0.1:5001/api` dan konfigurasi Database sesuai MySQL lokal Anda.

4.  Generate Key & Migrasi:
    ```bash
    php artisan key:generate
    php artisan migrate
    ```

5.  Build Assets & Jalankan Server:
    ```bash
    npm run build
    php artisan serve --port 8000
    ```

---

## 🤖 Konfigurasi Model LLM

### Menggunakan Google Gemini (Cloud)
1.  Dapatkan API Key dari [Google AI Studio](https://aistudio.google.com/).
2.  Set `.env`:
    ```env
    LLM_PROVIDER=gemini
    GEMINI_API_KEY=AIzaSy...
    GEMINI_MODEL=gemini-2.0-flash
    ```

### Menggunakan Ollama (Lokal)
1.  Install [Ollama](https://ollama.ai).
2.  Pull model, misal Llama 3 atau Qwen:
    ```bash
    ollama pull llama3.2:latest
    ollama pull qwen3:8b
    ```
3.  Jalankan Ollama Server: `ollama serve`.
4.  Set `.env`:
    ```env
    LLM_PROVIDER=ollama
    # Jika pakai Docker, gunakan host.docker.internal
    OLLAMA_BASE_URL=http://localhost:11434 
    OLLAMA_MODEL=llama3.2:latest
    ```

---

## 📚 Fitur Utama

### 1. Chat dengan Referensi
- Bot menjawab berdasarkan dokumen akademik yang diupload.
- Menampilkan sitasi/sumber di bawah jawaban.
- **[BARU]** Klik chip sumber untuk mendownload/membuka dokumen asli.
- **[BARU]** Sumber referensi tersimpan di history chat (persisten).

### 2. Manajemen Dokumen
- Upload file PDF/DOCX via Admin Panel.
- Otomatis indexing ke Vector Database (ChromaDB).
- "Reindex" untuk memperbarui knowledge base tanpa hapus data.

---

## ❓ Troubleshooting Umum

**Q: Chat bot menjawab "Saya tidak tahu"...**
A: Pastikan dokumen sudah diupload dan statusnya "Indexed" di Admin Panel. Cek log API untuk melihat apakah retrieval mengembalikan hasil.

**Q: Tidak bisa konek ke Database (Connection Refused)...**
A: Jika pakai Docker, host database adalah `mysql` (nama service). Jika manual, host adalah `127.0.0.1`. Cek password di `.env`.

**Q: Error `ModuleNotFoundError: No module named 'rag_model'`...**
A: Pastikan submodule `rag-model` sudah terdownload. Jalankan `git submodule update --init --recursive`.

---

## 🔄 Workflow Development (Polyrepo)

Sistem ini menggunakan **Git Submodules**. Berikut cara kerja / workflow jika Anda ingin melakukan perubahan kode:

### 1. Update Semua Repository
Selalu lakukan ini sebelum mulai bekerja untuk memastikan sinkronisasi.
```bash
git pull --recurse-submodules
git submodule update --remote --recursive
```

### 2. Melakukan Perubahan (Commit & Push)
Misal Anda mengubah kode di `rag-web`:

1.  **Masuk ke folder submodule:**
    ```bash
    cd rag-web
    ```
2.  **Lakukan perubahan, commit, dan push:**
    ```bash
    git add .
    git commit -m "Update fitur chat"
    git push origin main
    ```
3.  **Update referensi di Host Repo (`rag-deploy`):**
    Kembali ke root dan commit perubahan pointer submodule.
    ```bash
    cd ..
    git add rag-web
    git commit -m "Update rag-web submodule reference"
    git push origin main
    ```

### 3. Setup Ulang (Troubleshooting)
Jika struktur berantakan:
```bash
git submodule deinit -f .
git submodule update --init --recursive
```

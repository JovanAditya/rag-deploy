# RAG Deploy

Orchestration repository untuk sistem RAG Akademik Universitas Mercu Buana.

## 📋 Deskripsi

Repository ini berisi:
- **Docker Compose**: Deployment semua services
- **Submodules**: Link ke semua repository
- **Konfigurasi**: Environment variables

## 📁 Struktur Direktori

```
rag-deploy/
├── rag-model/              # Submodule
├── rag-api/                # Submodule
├── rag-web/                # Submodule
├── data/                   # Shared data
├── docker-compose.yml
├── docker-compose.dev.yml
├── .env.example
└── README.md
```

## 📚 Panduan Lengkap

Silakan baca **[COMPREHENSIVE_GUIDE.md](COMPREHENSIVE_GUIDE.md)** untuk instruksi detail mengenai:
- Instalasi via Docker vs Manual
- Konfigurasi Local LLM (Ollama, Qwen, Llama)
- Fitur-fitur utama

## ⚙️ Quick Start (Docker)

```bash
# Clone dengan submodules
git clone --recurse-submodules https://github.com/JovanAditya/rag-deploy.git
cd rag-deploy

# Konfigurasi
cp .env.example .env
# Edit .env: Pilih LLM_PROVIDER (gemini/ollama/dll) dan isi API Key

# Start services
docker-compose up -d --build
```

## 🔧 Konfigurasi

Edit file `.env` di root folder ini. File ini mengontrol seluruh konfigurasi stack (Frontend + Backend + Database).

Supported LLMs:
- **Cloud**: Gemini, OpenAI, Anthropic
- **Local**: Ollama (Llama 3, Qwen, Mistral, dll)

## 🐳 Services

| Service | Port | Deskripsi |
|---------|------|-----------|
| rag-api | 5001 | REST API |
| rag-web | 8000 | Laravel App |
| mysql | 3306 | Database |

## 📋 Perintah Docker

```bash
# Start
docker-compose up -d

# Stop
docker-compose down

# Rebuild
docker-compose up -d --build

# Logs
docker-compose logs -f rag-api
```

## 🔗 Git Submodules

```bash
# Update semua submodules
git submodule update --remote --recursive

# Pull dengan submodules
git pull --recurse-submodules
```

## 🔗 Repository Terkait

| Repository | Deskripsi |
|------------|-----------|
| [rag-model](https://github.com/JovanAditya/rag-model) | Core RAG Model |
| [rag-api](https://github.com/JovanAditya/rag-api) | REST API |
| [rag-web](https://github.com/JovanAditya/rag-web) | Laravel Frontend |

---

*Bagian dari proyek skripsi Sistem RAG Akademik - Universitas Mercu Buana*

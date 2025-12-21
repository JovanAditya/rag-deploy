# Panduan Git Workflow - Polyrepo dengan Submodules

Panduan lengkap untuk mengelola repository RAG Akademik yang menggunakan struktur polyrepo dengan Git Submodules.

---

## 📁 Struktur Repository

```
rag-deploy/          (Host/Parent Repository)
├── rag-model/       (Submodule - Core RAG)
├── rag-api/         (Submodule - REST API)
│   └── rag-model/   (Nested Submodule)
├── rag-web/         (Submodule - Laravel Frontend)
└── compose.yml
```

---

## 🚀 Clone Repository (Pertama Kali)

```bash
# Clone dengan semua submodules sekaligus
git clone --recurse-submodules https://github.com/JovanAditya/rag-deploy.git
cd rag-deploy
```

**Jika sudah clone tapi submodule kosong:**
```bash
git submodule update --init --recursive
```

---

## 🔄 Sinkronisasi (Sebelum Mulai Kerja)

**WAJIB dilakukan setiap kali mulai coding:**

```bash
cd rag-deploy

# Pull semua perubahan (parent + submodules)
git pull --recurse-submodules

# Update submodule ke commit terbaru
git submodule update --remote --recursive
```

---

## ✏️ Workflow: Mengubah Kode

### Langkah 1: Masuk ke Submodule yang Ingin Diubah

```bash
# Contoh: mengubah rag-web
cd rag-deploy/rag-web
```

### Langkah 2: Pastikan di Branch yang Benar

```bash
# Cek branch saat ini
git branch

# Jika dalam "detached HEAD", checkout ke main
git checkout main

# Pull perubahan terbaru
git pull origin main
```

### Langkah 3: Edit Kode, Commit, dan Push

```bash
# Edit file...

# Commit
git add .
git commit -m "fix: deskripsi perubahan"

# Push ke remote
git push origin main
```

### Langkah 4: Update Referensi di Parent (rag-deploy)

**PENTING:** Setiap kali push submodule, Anda harus update referensinya di parent.

```bash
# Kembali ke rag-deploy
cd ..

# atau
cd /path/to/rag-deploy

# Tambahkan perubahan submodule
git add rag-web  # (atau rag-api, rag-model)

# Commit referensi baru
git commit -m "chore: update rag-web submodule"

# Push parent
git push origin main
```

---

## ⚠️ Troubleshooting

### 1. Detached HEAD State

**Gejala:** `HEAD` tidak menunjuk ke branch manapun.

```bash
# Cek apakah detached
git branch
# Output: * (HEAD detached at abc1234)
```

**Solusi:**
```bash
# Pindahkan main ke HEAD saat ini
git branch -f main HEAD

# Checkout ke main
git checkout main

# Push
git push origin main
```

### 2. Push Ditolak (Non-Fast-Forward)

**Gejala:** Error "Updates were rejected because a pushed branch tip is behind"

**Solusi Aman (Rebase):**
```bash
git fetch origin
git rebase origin/main
git push origin main
```

**Solusi Cepat (Force Push - HATI-HATI):**
```bash
git push origin main --force
```

### 3. Submodule Tidak Sinkron

**Gejala:** Submodule menunjuk ke commit lama.

```bash
# Update semua submodule ke commit terbaru
git submodule update --remote --recursive

# Commit perubahan referensi
git add .
git commit -m "chore: sync all submodules"
git push origin main
```

### 4. Konflik Merge di Submodule

```bash
cd rag-web  # atau submodule lain

# Lihat status
git status

# Selesaikan konflik di file yang bertanda
# Edit file, hapus marker <<<<< ===== >>>>>

# Setelah selesai
git add .
git rebase --continue

# Push
git push origin main
```

---

## 📋 Urutan Push yang Benar

Selalu push dari **dalam ke luar** (submodule dulu, baru parent):

```
1. rag-model    → push
2. rag-api      → push (jika ada perubahan di rag-api/rag-model, update dulu)
3. rag-web      → push
4. rag-deploy   → update referensi semua submodule → push
```

**Contoh Lengkap:**

```bash
# === PUSH RAG-MODEL ===
cd rag-deploy/rag-model
git add . && git commit -m "feat: fitur baru"
git push origin main

# === PUSH RAG-API ===
cd ../rag-api
# Update nested submodule jika perlu
git add rag-model
git add . && git commit -m "feat: endpoint baru"
git push origin main

# === PUSH RAG-WEB ===
cd ../rag-web
git add . && git commit -m "fix: perbaikan UI"
git push origin main

# === PUSH RAG-DEPLOY (PARENT) ===
cd ..
git add rag-model rag-api rag-web
git commit -m "chore: update all submodules"
git push origin main
```

---

## 🔧 Perintah Git Submodule Berguna

| Perintah | Fungsi |
|----------|--------|
| `git submodule status` | Lihat status semua submodule |
| `git submodule update --init` | Inisialisasi submodule |
| `git submodule update --remote` | Update ke commit terbaru |
| `git submodule foreach git pull origin main` | Pull semua submodule |
| `git submodule foreach git status` | Cek status semua submodule |

---

## 💡 Tips

1. **Jangan edit submodule langsung dari parent** - Selalu `cd` ke folder submodule dulu.
2. **Selalu cek branch** - Pastikan tidak dalam detached HEAD sebelum commit.
3. **Commit message yang jelas** - Gunakan format: `type: deskripsi` (feat, fix, chore, docs, style).
4. **Pull sebelum push** - Hindari konflik dengan `git pull --rebase` sebelum push.
5. **Backup sebelum force push** - Buat branch backup jika tidak yakin.

---

*Dokumen ini dibuat untuk mempermudah pengelolaan repository RAG Akademik UMB.*

---
title: "Misteri Sesi Chat AI yang Hilang: Bug 'Split-Brain' Remote-SSH & Cara Menyelamatkannya"
description: "Mengupas tuntas bug ghost session pada Antigravity saat menggunakan arsitektur VS Code Remote SSH dan cara recover context dengan 2 prompt sakti."
date: 2026-03-31T10:00:00+07:00
author: "Sandi"
categories: ["DevOps", "AI"]
tags: ["ai", "vscode", "antigravity", "remote-ssh"]
image: "/images/blog/ai_split_brain_thumbnail.png"
draft: false
---

# Misteri Sesi Chat AI yang Hilang: Mengungkap Bug "Split-Brain" pada IDE

Pernahkah kalian mengobrol panjang lebar dengan AI asisten koding kalian, merencanakan deployment platform yang masif, lalu keesokan harinya—*puff!*—semua log perbincangan itu lenyap tanpa jejak dari UI? Ya, inilah yang baru saja membuat saya muak dan cukup membuat stres hari ini.

Ceritanya, saya sedang sibuk-sibuknya melakukan konfigurasi kritis: **"Deploying Naikkelas Production Environment"**. Mulai dari menyiapkan `docker-compose.yml`, men-setup *Nginx reverse proxy* untuk *domain routing*, hingga melakukan integrasi *database* Turso untuk platform Naikkelas. Semua diskusi, *troubleshooting*, dan perintah terminal dieksekusi dengan brilian bersama asisten AI saya (Antigravity).

Tetapi ketika saya melakukan *reload window*... opsi untuk membuka kembali sesi krusial itu tidak bisa diakses sama sekali! Di menu *Past Conversations*, saya hanya disuguhkan satu sesi "hantu" dari kemarin (*Modernizing Cinematic Learning Protocol*) yang ditandai dengan titik hitam (`•`) alias berstatus *corrupted*/*unsaved*. Sementara obrolan *Deployment* yang baru ditutup beberapa menit lalu lenyap tak tersisa.

## Mengungkap Biang Keroknya: Arsitektur "Split-Brain" pada Remote-SSH

Setelah *ngulik* sistem dan menyelediki folder internal `~/.gemini/antigravity/brain/`, saya akhirnya menemukan jawaban teknis mengapa *bug* menyebalkan ini bisa menjangkiti IDE kita, terutama bagi *developer* yang bekerja menggunakan lingkungan **VS Code Remote-SSH**.

Masalah utamanya ada pada apa yang saya sebut sebagai arsitektur **"Split-Brain" (Otak Terbelah)**.

Saat ekstensi AI berjalan di environment *Remote-SSH*, ia sejatinya beroperasi di dua dunia yang terpisah:
1. **UI Extension Host (Lokal di Laptop Anda)**: Bagian ini merender *chat bubble*, tombol antarmuka, dan menyimpan remah-remah *cache* (seperti daftar *History Dropdown*) secara lokal di memori laptop komputasi Anda.
2. **Workspace Extension Host (Berjalan di Server SSH)**: Ini adalah "otak" mesin *backend* sesungguhnya. Ia bertugas membaca *repository*, mengeksekusi terminal *bash*, dan menyimpan rakam jejak sesi (*artefak* historis) ke dalam *disk* server Linux Anda yang sebenarnya.

### Titik Terjadinya Bug (Desync)
Masalah sinkronisasi muncul akibat *network jitter* atau *reload* IDE sekian milidetik yang memutus transmisi antara Laptop dan Server.

- **Sisi Server SSH** sebetulnya berhasil merekam dan menulis *log* percakapan serta dokumen perencanaan kita dengan rahasia ke dalam berkas `implementation_plan.md` di folder `~/.gemini/antigravity/brain/<id-sesi>/`. File-file tersebut tersimpan **utuh dan sangat aman**.
- **Namun di sisi UI Laptop**, putusnya transmisi membuat ekstensi AI **gagal menangkap sinyal sinkronisasi** dari *server*. *Interface* UI di laptop Anda tidak mendata direktori baru tersebut ke dalam katalog *Dropdown History*. Ini membuat *chat* Anda seolah menguap di udara!

Sebaliknya, pada kasus *Ghost Session* masa lalu (kasus sesi "Cinematic" saya), UI di laptop masih mendata "*Saya ingat sesi Cinematic ini ada*", jadi labelnya muncul di menu. Tapi karena sesinya dihapus atau dibatalkan sepihak di server, UI hanya bisa menampilkan "titik hitam" (`•`) sebagai tanda *broken link* atau berkas *corrupt*.

Secara ringkas: **Ekstensi ini mengandalkan _State Memory_ lokal alih-alih melakukan pengecekan (*Force Polling*) secara langsung ke Storage Server setiap kali di-_reload_. Hal ini jelas merupakan kelemahan mutlak (_bug_) pada desain *user experience*!**

---

## Solusi & Cheat Sheet AI Prompts: Menyelamatkan Sesi Anda

Tentu saja *bug* ini membuat kesal, tetapi sadarlah: **Konteks pekerjaan Anda belum mati!** 

Karena fail-fail percakapan dan *plan* otomatis tersimpan aman di `/brain/` dalam bentuk berkas berakhiran ekstensi Markdown (`.md`), Anda bisa menyuruh AI di *sesi yang baru* untuk mencari dan mengingat kembali sesi yang hilang tadi. 

Berikut adalah **2 Template Prompt "Anti-Panik"** yang wajib Anda masukkan pada *New Session* ketika Anda kehilangan *history*:

### 1. Mencari Tahu ID dan Sesi Terbaru yang "Nyasar"
Gunakan *prompt* ini agar AI di sesi baru membaca manual isi berkas server Anda dan melaporkan sesi mana saja yang barusan Anda kerjakan hari ini:

> **Prompt:** 
> _"Tolong operasikan pencarian di server. Coba cari, baca metadata, dan tampilkan *list* semua session chat Antigravity di direktori `/home/dev/.gemini/antigravity/brain/` (atau direktori storage AI standar sistem ini) yang terakhir kali dimodifikasi dalam 1 jam hingga 1 hari terakhir. Tampilkan secara lengkap beserta tanggal, ringkasan judul proyeknya, dan berikan ID spesifik foldernya kepadaku."_

Setelah *prompt* ini dieksekusi, AI biasanya akan memaparkan tabel ID folder (misal: `14aebb83-...`) lengkap dengan tanggalnya. Begitu Anda mengenali sesi "Deployment Naikkelas" Anda dari *list* tersebut, salin ID-nya dan lanjut ke langkah 2.

### 2. Memaksa Sesi Baru Meng-Copy Konteks Sesi Lama (Amnesia Recovery)
Gunakan instruksi ini untuk membimbing AI di sesi baru agar bersikap seolah ia adalah iterasi dari percakapan Anda di sesi yang hilang tadi dengan mempelajari dokumen kerjanya:

> **Prompt:**
> _"Saya mengalami putus koneksi dan sesi kita sebelumnya hilang dari UI. Tolong gunakan *tool* baca berkasmu untuk membaca dengan saksama dan mempelajari secara menyeluruh isi dari file `implementation_plan.md`, `task.md`, serta `walkthrough.md` di dalam direktori `~/.gemini/antigravity/brain/<MASUKKAN_ID_SESI_YANG_HILANG_DI_SINI>/`. Pahami seluruh konteks historis, arsitektur, dan rencana kodingan kita sebelumnya di sana agar kita bisa melanjutkan progres persis dari titik terakhir pekerjaan tersebut secara mulus di sesi ini."_

Alhasil, sang asisten seketika kembali sepintar pada sesi awal, dan semua daftar ceklis tugas (kodingan/konfigurasi) yang tertunda bisa segera dilanjutkan bak tidak pernah ada insiden apapu!

## Hikmah Terpenting
*Bugs will be bugs.* Jangan pernah memasrahkan 100% *memory management project* Anda pada menu *History Extension IDE*. Selalulah menuntut AI Anda untuk merangkum hasil *troubleshooting/deployment plan* menjadi "Artefak Dokumentasi" (`.md`) yang di-*commit* ke repositori lokal Anda sendiri, karena meskipun memori RAM itu *"fragile"*, **Markdown (*Local File*) is Forever**.

*Keep coding, keep ranting, and backup your AI logs via Bash!*

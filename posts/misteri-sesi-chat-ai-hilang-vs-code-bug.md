---
title: "Misteri Sesi Chat AI yang Hilang: Bug 'Split-Brain' Remote-SSH & Cara Menyelamatkannya"
description: "Mengupas tuntas bug ghost session pada Antigravity saat menggunakan arsitektur VS Code Remote SSH dan cara recover context dengan 2 prompt sakti."
date: 2026-03-31T10:00:00+07:00
author: "Sandikodev"
categories: ["DevOps", "AI"]
tags: ["ai", "vscode", "antigravity", "remote-ssh"]
image: "/images/blog/ai_split_brain_thumbnail.png"
ai_features:
  mermaid_diagrams: false
  code_snippets: true
  technical_depth: "intermediate"
content_type: "technical"
auto_toc: true
reading_time: true
draft: false
---

# Misteri Sesi Chat AI yang Hilang: Mengungkap Bug "Split-Brain" pada IDE

Pernahkah kalian mengobrol panjang lebar dengan AI asisten koding kalian, merencanakan deployment platform yang masif, lalu keesokan harinya—_puff!_—semua log perbincangan itu lenyap tanpa jejak dari UI? Ya, inilah yang baru saja membuat saya muak dan cukup membuat stres hari ini.

Ceritanya, saya sedang sibuk-sibuknya melakukan konfigurasi kritis: **"Deploying Naikkelas Production Environment"**. Mulai dari menyiapkan `docker-compose.yml`, men-setup _Nginx reverse proxy_ untuk _domain routing_, hingga melakukan integrasi _database_ Turso untuk platform Naikkelas. Semua diskusi, _troubleshooting_, dan perintah terminal dieksekusi dengan brilian bersama asisten AI saya (Antigravity).

Tetapi ketika saya melakukan _reload window_... opsi untuk membuka kembali sesi krusial itu tidak bisa diakses sama sekali! Di menu _Past Conversations_, saya hanya disuguhkan satu sesi "hantu" dari kemarin (_Modernizing Cinematic Learning Protocol_) yang ditandai dengan titik hitam (`•`) alias berstatus _corrupted_/_unsaved_. Sementara obrolan _Deployment_ yang baru ditutup beberapa menit lalu lenyap tak tersisa.

## Mengungkap Biang Keroknya: Arsitektur "Split-Brain" pada Remote-SSH

Setelah _ngulik_ sistem dan menyelediki folder internal `~/.gemini/antigravity/brain/`, saya akhirnya menemukan jawaban teknis mengapa _bug_ menyebalkan ini bisa menjangkiti IDE kita, terutama bagi _developer_ yang bekerja menggunakan lingkungan **VS Code Remote-SSH**.

Masalah utamanya ada pada apa yang saya sebut sebagai arsitektur **"Split-Brain" (Otak Terbelah)**.

Saat ekstensi AI berjalan di environment _Remote-SSH_, ia sejatinya beroperasi di dua dunia yang terpisah:

1. **UI Extension Host (Lokal di Laptop Anda)**: Bagian ini merender _chat bubble_, tombol antarmuka, dan menyimpan remah-remah _cache_ (seperti daftar _History Dropdown_) secara lokal di memori laptop komputasi Anda.
2. **Workspace Extension Host (Berjalan di Server SSH)**: Ini adalah "otak" mesin _backend_ sesungguhnya. Ia bertugas membaca _repository_, mengeksekusi terminal _bash_, dan menyimpan rakam jejak sesi (_artefak_ historis) ke dalam _disk_ server Linux Anda yang sebenarnya.

### Titik Terjadinya Bug (Desync)

Masalah sinkronisasi muncul akibat _network jitter_ atau _reload_ IDE sekian milidetik yang memutus transmisi antara Laptop dan Server.

- **Sisi Server SSH** sebetulnya berhasil merekam dan menulis _log_ percakapan serta dokumen perencanaan kita dengan rahasia ke dalam berkas `implementation_plan.md` di folder `~/.gemini/antigravity/brain/<id-sesi>/`. File-file tersebut tersimpan **utuh dan sangat aman**.
- **Namun di sisi UI Laptop**, putusnya transmisi membuat ekstensi AI **gagal menangkap sinyal sinkronisasi** dari _server_. _Interface_ UI di laptop Anda tidak mendata direktori baru tersebut ke dalam katalog _Dropdown History_. Ini membuat _chat_ Anda seolah menguap di udara!

Sebaliknya, pada kasus _Ghost Session_ masa lalu (kasus sesi "Cinematic" saya), UI di laptop masih mendata "_Saya ingat sesi Cinematic ini ada_", jadi labelnya muncul di menu. Tapi karena sesinya dihapus atau dibatalkan sepihak di server, UI hanya bisa menampilkan "titik hitam" (`•`) sebagai tanda _broken link_ atau berkas _corrupt_.

Secara ringkas: **Ekstensi ini mengandalkan _State Memory_ lokal alih-alih melakukan pengecekan (_Force Polling_) secara langsung ke Storage Server setiap kali di-_reload_. Hal ini jelas merupakan kelemahan mutlak (_bug_) pada desain _user experience_!**

---

## Solusi & Cheat Sheet AI Prompts: Menyelamatkan Sesi Anda

Tentu saja _bug_ ini membuat kesal, tetapi sadarlah: **Konteks pekerjaan Anda belum mati!**

Karena fail-fail percakapan dan _plan_ otomatis tersimpan aman di `/brain/` dalam bentuk berkas berakhiran ekstensi Markdown (`.md`), Anda bisa menyuruh AI di _sesi yang baru_ untuk mencari dan mengingat kembali sesi yang hilang tadi.

Berikut adalah **2 Template Prompt "Anti-Panik"** yang wajib Anda masukkan pada _New Session_ ketika Anda kehilangan _history_:

### 1. Mencari Tahu ID dan Sesi Terbaru yang "Nyasar"

Gunakan _prompt_ ini agar AI di sesi baru membaca manual isi berkas server Anda dan melaporkan sesi mana saja yang barusan Anda kerjakan hari ini:

> **Prompt:**
> _"Tolong operasikan pencarian di server. Coba cari, baca metadata, dan tampilkan *list* semua session chat Antigravity di direktori `/home/dev/.gemini/antigravity/brain/` (atau direktori storage AI standar sistem ini) yang terakhir kali dimodifikasi dalam 1 jam hingga 1 hari terakhir. Tampilkan secara lengkap beserta tanggal, ringkasan judul proyeknya, dan berikan ID spesifik foldernya kepadaku."_

Setelah _prompt_ ini dieksekusi, AI biasanya akan memaparkan tabel ID folder (misal: `14aebb83-...`) lengkap dengan tanggalnya. Begitu Anda mengenali sesi "Deployment Naikkelas" Anda dari _list_ tersebut, salin ID-nya dan lanjut ke langkah 2.

### 2. Memaksa Sesi Baru Meng-Copy Konteks Sesi Lama (Amnesia Recovery)

Gunakan instruksi ini untuk membimbing AI di sesi baru agar bersikap seolah ia adalah iterasi dari percakapan Anda di sesi yang hilang tadi dengan mempelajari dokumen kerjanya:

> **Prompt:**
> _"Saya mengalami putus koneksi dan sesi kita sebelumnya hilang dari UI. Tolong gunakan *tool* baca berkasmu untuk membaca dengan saksama dan mempelajari secara menyeluruh isi dari file `implementation_plan.md`, `task.md`, serta `walkthrough.md` di dalam direktori `~/.gemini/antigravity/brain/<MASUKKAN_ID_SESI_YANG_HILANG_DI_SINI>/`. Pahami seluruh konteks historis, arsitektur, dan rencana kodingan kita sebelumnya di sana agar kita bisa melanjutkan progres persis dari titik terakhir pekerjaan tersebut secara mulus di sesi ini."_

Alhasil, sang asisten seketika kembali sepintar pada sesi awal, dan semua daftar ceklis tugas (kodingan/konfigurasi) yang tertunda bisa segera dilanjutkan bak tidak pernah ada insiden apapu!

## Hikmah Terpenting

_Bugs will be bugs._ Jangan pernah memasrahkan 100% _memory management project_ Anda pada menu _History Extension IDE_. Selalulah menuntut AI Anda untuk merangkum hasil _troubleshooting/deployment plan_ menjadi "Artefak Dokumentasi" (`.md`) yang di-_commit_ ke repositori lokal Anda sendiri, karena meskipun memori RAM itu _"fragile"_, **Markdown (_Local File_) is Forever**.

_Keep coding, keep ranting, and backup your AI logs via Bash!_

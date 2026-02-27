---
title: "Evolusi Arsitektur Web: Mengapa Kami Meninggalkan tRPC untuk Astro Actions"
description: "Sebuah narasi mendalam tentang perjalanan teknis project Roti Sholawat, dari era tRPC + TanStack Query hingga kesederhanaan Astro Actions + Svelte 5 Runes."
date: 2026-02-27
authors: ["sandikodev"]
categories: ["engineering", "architecture"]
tags: ["astro", "svelte", "trpc", "web-dev", "clean-code", "refactoring"]
image: "/images/posts/architecture-evolution.png"
draft: false
---

Dalam dunia pengembangan web yang bergerak secepat kilat, keputusan arsitektur hari ini bisa menjadi beban teknis hari esok. Artikel ini bukan tentang menyalahkan sebuah teknologi, melainkan tentang menghormati evolusi. Ini adalah kisah di balik layar project **Roti Sholawat**, sebuah sistem manajemen admin yang baru saja melewati transformasi arsitektur besar-besaran: dari tRPC ke Astro Actions.

## Prologue: Hari-hari tRPC yang Gemilang

Beberapa tahun lalu, jika Anda ingin membangun aplikasi _full-stack_ TypeScript yang aman, tRPC adalah standar emas. Ia menjanjikan sesuatu yang hampir mustahil di era REST API tradisional: **End-to-End Type Safety** tanpa _code generation_.

Di awal pengembangan Roti Sholawat, tRPC adalah "pahlawan" kami. Kami memiliki router backend yang kompleks untuk mengelola pesanan, invoice, dan inventaris. Setiap kali kami mengubah kolom di database (menggunakan Drizzle ORM), frontend Svelte kami langsung berteriak jika ada kode yang tidak kompatibel. Autocomplete meluncur mulus, produktivitas meningkat tajam.

Namun, seiring bertambahnya fitur, kami mulai merasakan aroma "Boilerplate Fatigue".

## Paradoks Boilerplate: Harga Sebuah Keamanan

Untuk membuat satu mutasi sederhana di tRPC, langkah-langkahnya cukup melelahkan:

1. Buat router di backend.
2. Definisikan skema Zod.
3. Registrasikan router ke root router.
4. Di frontend, gunakan `useQuery` atau `useMutation` dari TanStack Query.
5. Kelola state loading, error, dan success secara manual (lagi dan lagi).

Setup-nya terasa seperti memasang kabel telepon yang sangat panjang dan rumit hanya untuk berbicara dengan seseorang di ruangan yang sama. Kami menyadari bahwa meskipun tRPC memberikan keamanan, ia juga membawa beban kognitif yang besar melalui infrastruktur sekundernya.

Tentu saja, kami tidak menutup mata terhadap kehebatan tRPC. Kami menyadari penuh bahwa **tRPC memang benar-benar standar emas dan sangat ideal bilamana aplikasi yang dibangun adalah purni sebagai backend service**. Jika frontend-nya dirancang untuk didistribusikan dan dihubungkan ke berbagai platform _end-user_ lainnya—seperti aplikasi mobile dengan React Native, atau klien desktop pihak ketiga—dan semuanya dikelola dalam sebuah ekosistem _workspace monorepo_, maka tRPC adalah solusi yang luar biasa _powerful_. Ia memastikan semua platform tersebut berbicara dalam "bahasa" tipe data yang sama tanpa cacat.

Namun, untuk Roti Sholawat yang merupakan arsitektur terpadu (di mana frontend Astro/Svelte dan backend berada dalam satu _domain_ eksekusi yang sama), kekuatan _cross-platform_ tersebut tidak pernah benar-benar kami manfaatkan.

## Lahirnya Sang "RPC Killer": Astro Actions

Masuklah **Astro Actions**.

Astro 4.x dan 5.x memperkenalkan sebuah konsep yang sangat elegan dalam kesederhanaannya: **Native RPC**. Alih-alih membungkus permintaan dalam router eksternal, Astro memungkinkan kita mendefinisikan "Action" langsung sebagai fungsi server-side yang aman.

Perbandingannya mencengangkan:

- **tRPC**: Butuh setup router, `trpc.ts`, API endpoint khusus, dan library client.
- **Astro Actions**: Cukup tulis fungsi di `src/actions/index.ts`, dan Svelte bisa langsung memanggilnya melalui `actions.namaFungsi()`.

Kami mendapatkan keamanan tipe data (Type Safety) yang **SAMA PERSIS** dengan tRPC, namun dengan pemangkasan kode hingga 80%. Ini bukan sekadar pembersihan; ini adalah efisiensi murni.

## Svelte 5 Runes: Menutup Celah Terakhir

Alasan lain kami tetap bertahan di tRPC adalah integrasinya yang kuat dengan **TanStack Query**. Kemampuannya mengelola _synchronization state_ (loading, fetching, error) sangat membantu di Svelte 4.

Namun, **Svelte 5** datang merubah segalanya dengan **Runes**.
Dengan `$state` dan `$derived`, logika reaktif menjadi sangat ringan dan cerdas. Kami menyadari bahwa untuk dashboard admin CRUD, beban tambahan dari _caching layer_ TanStack Query seringkali lebih besar daripada manfaatnya.

Dengan Astro Actions yang menangani mutasi dan Svelte 5 yang menangani reaktivitas lokal, kami menemukan harmoni baru yang jauh lebih simpel.

## Perjalanan Migrasi: Fase A hingga D

Transformasi ini tidak terjadi dalam semalam. Kami membaginya menjadi fase-fase strategis untuk menjaga stabilitas:

- **Fase A - Katalog**: Kami memulai dengan modul paling sederhana (Kategori & Inventaris). Di sini kami memvalidasi bahwa Astro Actions benar-benar sekuat tRPC.
- **Fase B - Transaksional**: "Jantung" dari aplikasi. Kami memindahkan logika Orders, Invoices, dan Shipping. Di sinilah kompleksitas Drizzle ORM dan Zod diuji habis-habisan.
- **Fase C - Pemasaran**: Modul Coupon dan Ads dimodernisasi. Kami mengganti ribuan baris kode TanStack Query yang berbelit dengan Runes yang elegan.
- **Fase D - The Purge (Pembersihan Total)**: Ini adalah momen yang paling memuaskan.

## Phase D: Momen Katarsis Teknis

Malam ini, kami akhirnya mengeksekusi perintah yang menandai kedewasaan arsitektur kami:
`rm -rf src/server/routers src/server/trpc.ts`

Menghapus file-file tersebut bukan berarti kami membenci tRPC. Sebaliknya, itu adalah tanda bahwa kami telah mencapai tujuan yang ingin dicapai tRPC, namun melalui jalur yang lebih native dan terintegrasi dalam ekosistem Astro.

Aplikasi Roti Sholawat kini berjalan lebih ringan, _build time_ lebih cepat, dan yang terpenting: **Developer Experience (DX) meningkat drastis**. Tidak ada lagi file "kabel telepon" yang bergelantungan. Hanya kode murni yang fokus pada logika bisnis.

## Epilog: Belajar untuk Melepaskan

Pelajaran terbesar dari transformasi Roti Sholawat adalah: **Jangan takut untuk membuang apa yang pernah membantu Anda, jika sesuatu yang lebih baik telah hadir.**

tRPC diciptakan untuk mengisi lubang besar dalam komunikasi backend-frontend. Astro Actions datang dan menutup lubang itu secara permanen di tingkat framework. Sebagai engineer, tugas kita bukan untuk setia pada alat, melainkan setia pada efisiensi dan kemudahan _maintenance_.

Selamat tinggal tRPC, terima kasih atas jasa-jasamu. Selamat datang era "Zero Boilerplate" bersama Astro Actions dan Svelte 5.

---

_Ditulis dengan semangat clean code dan kopi malam, per 27 Februari 2026._

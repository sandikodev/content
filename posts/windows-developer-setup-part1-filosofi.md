---
title: "Windows untuk Developer, Part 1: Filosofi — Sanitized Host, CLI-First, dan Etika Digital"
description: "Setup yang bersih bukan soal estetika — ada alasan yang lebih dalam. Kenapa saya peduli dengan dirty host, kenapa CLI bukan gaya-gayaan, dan kenapa ini semua ujungnya nyambung ke etika."
date: 2026-04-05T18:30:00+07:00
author: "Sandikodev"
categories: ["DevOps"]
tags:
  [
    "windows",
    "linux",
    "philosophy",
    "cli",
    "ethics",
    "developer-workflow",
    "clean-setup",
    "richard-stallman",
  ]
image: "/images/blog/windows-clean-developer-philosophy.webp"
ai_features:
  mermaid_diagrams: false
  code_snippets: true
  technical_depth: "intermediate"
content_type: "technical"
auto_toc: true
reading_time: true
draft: false
---

_Ini adalah Part 1 dari series [Windows untuk Developer](/blog/windows-developer-setup-series-gateway). Kalau kamu belum baca gateway-nya, tidak apa-apa — artikel ini bisa berdiri sendiri._

---

## Dari Arch Linux ke Windows: Perjalanan yang Tidak Linear

Sebelum saya bicara tentang Windows, saya perlu cerita sedikit tentang perjalanannya — karena konteks ini penting untuk memahami mengapa perspektif saya berbeda dari kebanyakan artikel "setup Windows untuk developer" yang ada.

Saya tidak datang ke Windows karena tidak tahu yang lain. Saya datang setelah melewati:

- **Arch Linux** — distro yang memaksa kamu memahami setiap layer sistem dari bawah. Tidak ada installer yang memegang tanganmu. Kamu partisi disk sendiri, kamu konfigurasi bootloader sendiri, kamu pilih setiap paket yang masuk ke sistem.
- **Gentoo** — satu level lebih dalam. Di sini kamu tidak hanya memilih paket, tapi mengkompilasi semuanya dari source code dengan USE flags yang kamu tentukan sendiri.
- **Linux From Scratch (LFS)** — membangun sistem operasi dari nol. Bukan install, bukan konfigurasi — tapi benar-benar membangun, satu paket demi satu paket, dari kernel sampai userspace.

```
Timeline OS:
2015 → Arch Linux (belajar internal OS)
2018 → Ubuntu / Debian (produktivitas)
2020 → CentOS (enterprise server)
2022 → Windows 10/11 + WSL2 (era stabilitas)
```

Perjalanan ini bukan tentang "akhirnya menyerah ke Windows". Ini tentang memilih platform yang tepat untuk fase kehidupan yang tepat — setelah benar-benar memahami apa yang ada di balik setiap pilihan, bukan sekadar ikut arus komunitas yang sedang ramai membicarakan distro tertentu.

Justru karena latar belakang Linux yang dalam itulah saya bisa melihat dengan jelas: **apa yang Windows lakukan dengan baik, dan di mana ia perlu dibantu**.

---

## Masalah Struktural Windows: Jujur Dulu

Sebelum bicara solusi, kita harus jujur tentang masalahnya.

Linux dan Unix punya filosofi yang elegan: **everything is a file**. Konfigurasi? File teks di `/etc`. Process? File di `/proc`. Socket? File di `/var/run`. Semua terstruktur, semua bisa di-inspect dengan text editor biasa, semua bisa di-version control dengan Git.

Windows lahir dari dunia yang berbeda. DOS dirancang untuk **menyerap pasar multimedia** di era 80-90an, bukan untuk keindahan arsitektur. Hasilnya adalah warisan yang masih kita tanggung sampai sekarang:

```
Linux/Unix                    Windows
─────────────────────         ─────────────────────
/etc/nginx/nginx.conf    vs   HKEY_LOCAL_MACHINE\SOFTWARE\...
~/.bashrc                vs   %APPDATA%\...\settings.json
/usr/bin/python3         vs   C:\Python311\python.exe (+ PATH chaos)
systemd unit files       vs   Windows Services + Task Scheduler
```

Registry adalah warisan DOS-era yang bertahan bukan karena ia adalah ide yang bagus, tapi karena **kompatibilitas adalah raja** di ekosistem Windows. Setiap aplikasi yang kita install akan turut menuliskan konfigurasinya ke sana — path binary, lisensi, preferensi, sampai hal-hal yang bahkan tidak kamu minta. Setiap uninstall meninggalkan sisa entri yang tidak ikut terhapus. Bertahun-tahun kemudian, registry menjadi kuburan konfigurasi yang tidak ada yang berani sentuh — bahkan Microsoft sendiri.

Tapi — dan ini penting — Windows tetap bertahan di industri, bahkan mendominasi. Itu bukan kebetulan. Itu adalah bukti bahwa **pragmatisme mengalahkan purisme** di pasar massal — Windows berhasil karena ia memprioritaskan "bisa dipakai oleh semua orang" di atas "benar secara arsitektural", dan ternyata pasar lebih menghargai yang pertama.

Kita tidak perlu memilih antara mengagumi Linux atau menggunakan Windows — keduanya bisa hidup berdampingan, selama kita tahu kapan harus menggunakan yang mana.

---

## Dirty Host vs Sanitized Host

Ini adalah konsep inti dari seluruh filosofi ini. Setiap kali kita mau install sesuatu di Windows, ada satu pertanyaan yang harus dijawab dulu:

**"Apakah ini mengotori host saya?"**

| Karakteristik    | Dirty Host                                        | Sanitized Host                                      |
| ---------------- | ------------------------------------------------- | --------------------------------------------------- |
| Cara install     | `.exe` installer global, `pip install` tanpa venv | WSL2, Docker, atau Nix Shell                        |
| Dampak ke sistem | Registry kotor, PATH membengkak                   | Sistem induk tetap bersih dan cepat                 |
| Versioning       | Sulit jalankan dua versi DB sekaligus             | Mudah — setiap proyek punya container sendiri       |
| Uninstall        | Selalu meninggalkan residu di `AppData`           | Hapus container/folder, laptop kembali seperti baru |

Perbedaannya bukan hanya soal kerapian. Ini soal **konsekuensi jangka panjang**.

Laptop yang dirty host akan semakin lambat seiring waktu — bukan karena hardware-nya memburuk, tapi karena akumulasi dari ratusan keputusan instalasi yang masing-masing meninggalkan sedikit jejak. Scheduled tasks yang tidak pernah dihapus. Services yang autostart tapi tidak pernah dipakai. PATH yang sudah panjangnya tidak masuk akal. Semua itu tidak terasa di hari pertama, tapi enam bulan kemudian kita mulai bertanya-tanya kenapa laptop yang dulu kencang sekarang terasa berat.

Sanitized host adalah kebalikannya. Setiap tool development hidup di dalam container atau environment yang terisolasi — kalau tidak dibutuhkan lagi, dihapus bersih tanpa sisa.

**Software adalah teks.** Dan teks yang baik adalah teks yang terstruktur, bisa di-version control, dan bisa dihapus bersih. Filosofi ini berlaku untuk kode yang kita tulis, dan seharusnya berlaku juga untuk environment tempat kode itu hidup.

---

## CLI-First: Bukan Soal Terlihat Keren

Ada kesalahpahaman yang sering saya temui: orang mengira developer yang pakai terminal itu sedang pamer, atau sedang nostalgia dengan era komputer jadul, atau sekadar ingin terlihat keren.

Tidak. Ada alasan yang jauh lebih pragmatis.

**GUI menyembunyikan realitas.**

Tombol "Install" di installer Windows bisa menjalankan puluhan operasi sekaligus — menulis ke registry, membuat scheduled tasks, menginstall services, memodifikasi PATH, bahkan menambahkan startup entries — semua tanpa sepengetahuanmu. Kamu klik satu tombol, dan kita tidak tahu apa yang terjadi di baliknya.

**CLI memberikan transparansi penuh.**

```bash
# Kamu bisa baca script ini sebelum menjalankannya
curl -fsSL https://get.docker.com | sh

# Kamu lihat persis apa yang terjadi
# Tidak ada yang tersembunyi
```

Setiap perintah yang kamu ketik di terminal adalah manifestasi eksplisit dari kehendakmu. Kamu melihat argumen yang dikirim, permission yang ditolak, exit code yang jujur — tidak ada lapisan abstraksi yang menyembunyikan apa yang sebenarnya terjadi di balik layar.

Ini juga relevan untuk keamanan. Aplikasi GUI bisa mengeksekusi script background tanpa sepengetahuanmu — dan kita tidak akan tahu sampai sesuatu yang aneh terjadi, atau sampai kita iseng buka Process Monitor dan melihat ada proses yang tidak kita kenali berjalan diam-diam. Dalam CLI, kamu adalah operator: setiap proses yang berjalan adalah karena kamu yang secara eksplisit memintanya.

Filosofi intinya sederhana: **software bukanlah apa yang terlihat (GUI), melainkan apa yang tertulis (teks/command)**. GUI adalah lapisan presentasi — ia memudahkan pengguna awam, tapi ia juga menyembunyikan apa yang sebenarnya terjadi. Developer yang hanya nyaman dengan GUI adalah developer yang terbiasa melihat kulit tanpa pernah menyentuh isinya.

---

## Git Hygiene: Identitas di Era AI

Ini mungkin terdengar tidak berhubungan dengan "setup Windows yang bersih", tapi sebenarnya sangat terhubung — karena keduanya berakar dari nilai yang sama: **akuntabilitas**.

Di dunia yang sekarang dibanjiri kode generatif AI, pertanyaan "siapa yang menulis kode ini?" menjadi semakin penting. Siapapun bisa generate kode dengan nama dan email siapapun. Siapapun bisa membuat commit yang terlihat seperti dari kamu.

Kecuali kalau kamu menggunakan **GPG signing**.

```bash
# Setup GPG signing untuk semua commit
git config --global user.signingkey YOUR_GPG_KEY_ID
git config --global commit.gpgsign true

# Setiap commit sekarang punya tanda tangan digital
git log --show-signature
# commit abc123...
# gpg: Good signature from "Sandikodev <dev@sandikodev.com>"
```

Tanda centang "Verified" di GitHub bukan sekadar kosmetik. Ini adalah **proof of personal accountability** — bukti kriptografis bahwa commit tersebut benar-benar dari kamu, bukan dari orang yang memalsukan identitasmu, bukan dari bot, bukan dari AI yang berjalan atas nama orang lain.

Di era AI yang bisa generate dan commit kode atas nama siapapun, GPG signing adalah cara kamu menyatakan: _"Ini saya. Saya bertanggung jawab atas setiap baris ini."_

Soal commit message — ini bukan soal estetika:

```bash
# ❌ Tidak berguna
git commit -m "fix stuff"
git commit -m "update"

# ✅ Atomic commits yang bermakna
git commit -m "feat(auth): add JWT refresh token rotation"
git commit -m "fix(api): handle null response from payment gateway"
```

Atomic commits adalah tentang kemampuan **rollback dengan presisi**. Bukan rollback seluruh hari kerja, tapi rollback satu fitur spesifik yang bermasalah — tanpa menyentuh yang lain.

---

## Etika Digital: Di Mana Semua Ini Bermuara

Sampai di sini, kita sudah bicara tentang sanitized host, CLI-first, dan git hygiene. Semuanya terdengar teknis. Tapi ada benang merah yang menghubungkan semuanya, dan benang itu bukan teknis — ia **etis**.

Ini yang sejak lama digembor-gemborkan oleh **Richard Stallman** — pendiri GNU, pejuang software freedom. Bagi Stallman, pertanyaannya bukan hanya "apakah software ini bekerja?" tapi "apakah software ini menghormati kebebasan penggunanya?" Apakah ia transparan tentang apa yang ia lakukan? Apakah ia bisa diaudit? Apakah ia memberikan kontrol kepada pengguna, atau diam-diam mengambilnya?

Kamu tidak harus setuju dengan semua posisi Stallman yang kadang ekstrem. Tapi inti argumennya relevan: **software yang kita gunakan membentuk cara kita berpikir dan bekerja**. Software yang opaque, yang menyembunyikan apa yang ia lakukan, yang mengambil data tanpa consent — itu bukan hanya masalah privasi teknis. Itu masalah kedaulatan mental.

Ada pemikiran yang lebih ekstrem lagi dalam spektrum ini: **Terry Davis** dan **TempleOS**-nya. Davis membangun sistem operasi sendiri dari nol — seorang diri, selama bertahun-tahun — sebagai ekspresi murni dari keyakinan bahwa komputer seharusnya menjadi alat yang sepenuhnya dipahami dan dikontrol oleh penggunanya. Kisahnya tragis dan kompleks, tapi ada kernel kebenaran di dalamnya yang menarik bagi kalangan yang serius memikirkan hubungan manusia dengan mesin. _(Untuk yang penasaran: [Palladium Magazine punya ulasan yang menarik](https://www.palladiummag.com/2022/04/01/palladium-is-now-templeos/) tentang warisan Davis.)_

Kita tidak perlu ke ekstrem itu untuk mengambil pelajarannya — bahwa ada nilai dalam memahami sistem yang kita gunakan, bukan hanya menggunakannya.

### Mental adalah Software

Ini analogi yang saya pegang erat:

> **Mental manusia adalah software. Tubuh adalah hardware-nya.**

Software yang berjalan di atas hardware yang rusak akan corrupt. Tapi berlaku juga sebaliknya: hardware yang terus-menerus menjalankan software yang rusak, cepat atau lambat akan ikut rusak.

Persis seperti virus di komputer — software yang terjangkit tidak berhenti merusak dirinya sendiri. Ia **menyebar**. Ia menginfeksi proses lain, memodifikasi behavior sistem, dan pada akhirnya merusak seluruh ekosistem yang bersentuhan dengannya.

Kita sudah melihat ini terjadi secara masif dan terdokumentasi: **konten brainrot di TikTok dan platform sejenis**. Bukan sekadar konten yang tidak berguna — ini adalah konten yang secara algoritmik dirancang untuk memaksimalkan waktu tonton dengan cara paling murah: stimulasi instan, tanpa konteks, tanpa makna, tanpa konsekuensi. Nonsense yang merusak kemampuan fokus, mengurangi toleransi terhadap kompleksitas, dan secara perlahan mendegradasi kapasitas berpikir kritis penggunanya.

Ini bukan teori. Ini adalah **desain yang disengaja** — engagement metrics yang dioptimalkan tanpa mempertimbangkan dampak pada "hardware" penggunanya.

Muaranya selalu sama: **brainware yang terdegradasi**. Manusia yang semakin sulit membaca teks panjang. Semakin tidak sabar dengan penjelasan yang membutuhkan konteks. Semakin mudah diprovokasi oleh stimulus sederhana. Semakin sulit membedakan informasi yang valid dari yang tidak.

Software yang rusak, menyebar, dan merusak hardware-nya sendiri.

### Tanggung Jawab Kita sebagai Developer

Maka pertanyaannya bukan hanya "bagaimana cara setup environment yang bersih?" tapi juga:

- Software yang kamu bangun — apakah ia menghormati penggunanya?
- Data yang kamu kumpulkan — apakah penggunamu benar-benar memahami konsekuensinya?
- Sistem yang kita deploy — apakah ia transparan tentang apa yang ia lakukan?

User biasa sekalipun sebenarnya wajib memahami batasan privasi dalam etika dunia digital. Bukan karena mereka harus jadi programmer, tapi karena mereka adalah pengguna yang berinteraksi dengan sistem yang mempengaruhi kehidupan mereka setiap hari.

Kita, sebagai developer, punya tanggung jawab yang lebih besar lagi. Kita adalah orang-orang yang membangun sistemnya.

> **Sebaik-baik teknologi, semua kembali pada SDM itu sendiri.**

Setup yang bersih, environment yang terisolasi, commit yang ditandatangani — semua itu adalah ekspresi dari satu nilai yang sama: kesadaran bahwa setiap keputusan teknis adalah keputusan etis juga.

---

_Lanjut ke [Part 2: WSL2 Deep Dive](/blog/windows-developer-setup-part2-wsl2) untuk implementasi teknisnya — bagaimana filosofi ini diterjemahkan ke dalam setup WSL2 yang serius._

_Atau kembali ke [Series Gateway](/blog/windows-developer-setup-series-gateway) untuk memilih topik lain._

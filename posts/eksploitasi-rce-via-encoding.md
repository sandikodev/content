---
title: "Eksploitasi RCE via Encoding: Catatan Kelam Keamanan Siber Klasik"
description: "Bagaimana karakter terselubung (multibyte poison) sanggup mendobrak pelindung aplikasi sisi peladen, meloloskan perintah Terminal terlarang, dan membakar akses Remote Code Execution (RCE)."
date: 2026-02-21T21:20:00+07:00
author: "Sandikodev"
categories: ["Security", "Web Development"]
tags: ["RCE", "Encoding", "Cybersecurity", "Zero Day", "Backend"]
image: "/images/blog/encoding-rce.png"
draft: false
---

Kita telah mengawal betapa hancurnya pengalaman pengguna saat huruf bermutasi (*Mojibake*) dan bagaimana *browser* dapat menjadi *zombie* peretas (*Mutational XSS*). Tapi tahukah Anda ada tingkat ketiga yang jauh lebih destruktif?

Bukan hanya *Javascript* klien yang dikuasai. Kita berbicara mengenai ekskalasi total: server pusat *(Backend)* Anda jatuh ke genggaman. Layar hitam Terminal terbuka. Berkas rahasia tersalin. Mesin dieksploitasi sepenuhnya. Ini adalah arena yang disebut **Remote Code Execution (RCE)**.

Dan ya, ini bisa berawal murni karena Anda menyepelekan proses konversi *Character Encoding* dalam arsitektur aplikasi sisi-peladen (*Server-Side*).

### Ilusi Sanitasi dan Anatomi Karakter Majemuk (*Multibyte*)

Bayangkan *Backend Engineer* yang super teliti. Mereka menulis serentetan `Regex` (Regular Expression) gahar, dan berbagai aturan kebal seperti `str_replace()` atau `addslashes()` fungsi *escape* untuk mengurung dan melumpuhkan komando-komando aneh (*Shell Execution* / SQL *Injection*).

Sebagai contoh, kutip ganda/tunggal (`"`, `'`) yang jadi nyawa sebuah kueri perintah, selalu lolos diselamatkan dengan menempelkan karakter pelindung *backslash* (`\` atau heksa `%5c`) di depannya menjadi (`\'` / `\"`). Server dipandang kebal 100%.

Tapi bagaimana jika sistem pembersih (*escape logic*) tersebut hidup dalam dimensi Latin/Inggris-Sentris (`latin1` ASCII), sementara *parser* di ujung rantai (contohnya *Database* MySQL atau *Interpreter OS Shell*) dikonfigurasi menggunakan bahasa dunia yang luas (GBK Cina, Shift_JIS Jepang, atau UTF-8)?

Di sinilah *"Peracunan Karakter Tumpuk"* (*Multibyte Poisoning*) bekerja merobek lapisan logika perlindungan *Backend*.

### Anatomi Serangan

Dalam *Encoding* kuno Cina atau Jepang (seperti `GBK`), setiap teks dibangun tidak hanya dari satu balok heksa, melainkan dua unit balok *byte* berdampingan.

Sebagai studi kasus kelam di tahun-tahun eksploitasi PHP (`addslashes` vs GBK Bypass):
Seorang *Hacker* memasukkan teks `縗'` (*String Heksa:* `%bf%27`).

Fungsi *filter backend* (`addslashes/escape`) Anda segera mencium tanda bahaya di ujung teks: ada tanda kutip tunggal bahaya `'` (`%27`)! Logika ini dengan bodoh dan sigap segera menempelkan perisai tebal pahlawannya, yakni sang *backslash* `\` (`%5c`).

Kini tangkapan tersebut *diubah/ditumpuk* dalam lalu-lintas RAM server menjadi himpunan `%bf %5c %27`. *Escape* sukses! Kursor aman, bukan? 

**TIDAK.** 

Tepat di seberang sana, saat rangkaian data ini akhirnya dimakan dan diurai ulang oleh utilitas internal *shell/database* yang sudah dikondisikan membaca mode bahasa (misal dikonfigurasi MySQL menggunakan *charset* GBK), terjadilah hal magis yang menghancurkan struktur.

Modul bahasa (Multy-byte *decoder*) melihat sepasang blok `%bf` dan temannya (si karakter lolosan pelindung tadi) `%5c`. Di dalam otak *Dictionary* bahasa Timur, himpunan `(%bf%5c)` adalah kode terdaftar untuk merepresentasikan atau melahirkan sebuah alfabet sah bernama **"縗"**. 

Secara ajaib, pelindung sakti Anda (`\`) *diserap*, dilahap sebagai gabungan bagian abjad. Ia musnah!

Dan mari lirik ujung *string* terakhir itu sekarang... karakter sisanya adalah teman lamanya yang polos, murni, dan tak bertuan: **`%27`** alias karakter kutip telanjang **`'`**.

Pelindung luluh lantak, karakter *escape* ditelan mentah oleh terjemahan mesin ganda, dan karakter eksekusi perintah (seperti tanda petik kueri interaktif OS/Database) lolos bebas menyemprotkan deklarasi *Shell Execution/RCE/SQL-i*.

### Studi Kasus: "Satu Bita yang Menentukan Jatuhnya Kerajaan"

Salah satu ancaman klasiknya tertanam abadi pada pustaka C/C++ bahasa tingkat rendah hingga ekosistem Node.js, Ruby, & PHP *legacy* di masa keemasannya. Ketika pengembang membedakan standar `collation`/ *character set* antara lapisan Middleware dan Database (misal Web Frameworknya ANSI dan Databasenya GBK). 

Serangan ini memaksa *Engineer* kelas dunia tidak bisa lagi sekadar membuang masalah "*encoding error*" ke laci daftar *bug minor*.

Ini adalah alasan mutlak fungsi kuno digeser habis. PHP membunuh kepercayaan pada `addslashes` dan beralih ke `real_escape_string` (yang mempertimbangkan deklarasi bahasa per-Sesi SQL). Di ranah *Frontend/Islands Architecture* modern, inilah dorongan mengapa pengembang Qwik dan Astro *harus* memaksakan meta-charset di urutan baris `#1` agar tidak memancing *content guessing* sedari akar.

Siklus *Character Encoding* adalah urat nadi *Input/Output*. Mengartikulasikannya dengan sinkron dan eksplisit—dari ujung server (UTF-8 murni), lewat basis data, perantara transit (JSON), hingga baris kanvas HTML pertama *browser* klien (Meta *Charset*)—adalah satu-satunya cara tidur nyenyak di dunia *Modern Security Architecture*.

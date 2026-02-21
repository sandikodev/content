---
title: "Mutational XSS (mXSS): Ketika Browser Menjadi Musuh dalam Selimut"
description: "Mendalami ancaman mXSS di arsitektur aplikasi sisi-klien modern, saat DOM Tree bereksperimen dengan input tak terenkripsi yang meretas dirinya sendiri tanpa henti."
date: 2026-02-21T21:15:00+07:00
author: "Sandikodev"
categories: ["Security", "Web Development"]
tags: ["mXSS", "DOM", "Security", "Browser Behavior", "Client-Side"]
image: "/images/blog/mutational-xss.png"
draft: false
---

Dalam rentang *Cybersecurity*, serangan *Cross-Site Scripting* (XSS) adalah ancaman menahun. Ketika seorang *Backend Engineer* membersihkan *string* `<script>` atau tag asing dari basis data, mereka berharap segalanya steril.

Tetapi masalah mendasar muncul dari tempat yang seharusnya paling terlindungi: **Browser Sang Pengguna**. 

Inilah evolusi mengerikan dari anomali pembacaan *Character Encoding* ke wilayah *JavaScript modern (Client-Side)*. Ia disebut **Mutational XSS (mXSS)**. Celah ini muncul tidak sekadar karena *hacker* menyusupkan *script* yang agresif, melainkan berawal dari misinterpretasi sepele *browser*—sebuah mutasi yang menghancurkan struktur DNA.

### Miskulturasi DOM (Document Object Model)

Dalam pengembangan modern, platform SPA (React, Vue, Qwik, dll) berinteraksi cepat dengan *Data Mentah* (Raw String), melemparnya ke dalam *Document Object Model* (DOM) yang lincah menggunakan teknik manipulasi berisiko seperti `innerHTML`, atau bahkan lewat injeksi API bawaan tanpa *wrapper* tipe data aman (DOMPurify).

Browser (sebut saja Chrome, Safari, atau Firefox), adalah *"parser"* HTML/XML super kompleks. Tugas alamiah *browser* ketika menerima sepotong *syntax* kotor atau setengah-setengah (*malformed*) adalah memperbaikinya! 

Iya, *browser* suka mencoba menjadi pahlawan yang merapikan HTML yang cacat (*mangled code*). Namun di titik perbaikan sepihak inilah letak bencana kemutasi (*Mutational*) itu bermula.

### Kronologi Manipulasi mXSS

Bagaimana jika Anda memberikan *input string* yang bersih dari tag berbahaya `<script>`, tetapi penuh sesak dengan kepingan atribut tak wajar dan deklarasi *Encoding* yang membingungkan *engine DOM* (seperti `&amp;` dan tanda petik gantung)?

```html
<img src="valid.jpg" title="&quot;&lt;img src=x onerror=alert(1)&gt;">
```

Perhatikan: Teks `<img src=x onerror=alert(1)>` disembunyikan dalam wujud entitas kode HTML murni (`&quot;&lt;img...`) dan *dikubur utuh secara aman* di dalam variabel tunggal `title=".."`. Sebuah Filter pelacak akan menyepakati bahwa tidak ada tag fungsional yang bisa dieksekusi dari baris teks konyol tersebut. Itu adalah atribusi *caption* yang suci!

**Teror Dimulai:** 
Namun *Hacker* memasukkan kode ini ke sebuah kerangka kerja modern (katakanlah lewat *chat history* atau interaksi forum) di mana komponen HTML membaca ulang *DOM Tree* milik laman (misalnya saat fitur editor mengubah *preview Mode*, atau sistem sinkronisasi mencoba mengambil `.innerHTML` yang ada lalu menggabung dan menulisnya ulang menjadi komponen yang lebih besar).

Ketika DOM lama dibaca ulang (`element.innerHTML`), *parser* super-*heroic* milik sang peramban mencoba "menormalisasi" kembali barisan tanda baca entitas tersebut.

Browser secara diam-diam *mendekode* ulang entitas HTML `&quot;...&quot;` dan `<...>` agar menjadi teks manusia yang *"rapi"*. Tapi masalahnya, perbaikannya mengubah identitas string tersebut! 

Setelah pemrosesan ganda, keluaran mutasi barunya kini menyemburkan *markup* telanjang:
```html
<img src="valid.jpg" title=""><img src=x onerror=alert(1)>"
```

Celah itu robek dari dalam. Tag `<img src=x>` berhasil lepas sepenuhnya dari penjara `title`. Berdiri bebas sebagai elemen kedua. Karena letak gambarnya tidak ada (`x`), argumen komando `onerror=...` menyala seketika dan mengambil alih eksekusi *JavaScript* sistem Anda.

### Benang Merah dengan *Character Encoding*

Mutasi mengerikan di atas bertindak jauh lebih radikal saat aplikasi gagal mendeklarasikan secara mantap wewenang charset lokal (*Unicode/UTF-8*) di permulaan `<head>`. 

Peretas cerdas bisa memborbardir celah mXSS dengan mengirim puluhan representasi *multibyte* langka dari bahasa asing eksotis. Sebuah karakter Cina tak terbaca, bila dipaksa dikompres dan diekspansi bolak-balik dalam variabel JavaScript (`JSON.stringify`/`parse`), kadang merontokkan tanda pembatas fungsi `}}` jika *encoding backend* dan *frontend browser* tak selaras.

Abaikan `meta charset`, dan Anda membiarkan *browser*—senjata pertahanan terakhir aplikasi web sisi-klien Anda—menusuk Anda dari kerah belakang lewat tangan pembersih otomatisnya.

Solusinya? Paksa `<meta charset="utf-8" />` di baris pertama `<head>` dan hindari *baking-in* atribut dinamis input mentah langsung ke dalam instrumen HTML parsial. Selalu manfaatkan standar API *Sanitizer* bawaan kerangka kerja (seperti Node ganti `.innerText`/kerjakan AST mapping). Jangan tantang ketekunan murni dari *engine Mutation* milik browser.

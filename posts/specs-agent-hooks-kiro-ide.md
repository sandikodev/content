---
title: "Specs dan Agent Hooks di Kiro IDE: Cara Baru Kerja Bareng AI"
description: "Bukan sekadar autocomplete. Specs dan hooks mengubah cara kamu merencanakan, mengeksekusi, dan menjaga kualitas kode bersama AI agent."
date: 2026-04-01T10:00:00+07:00
author: "Sandikodev"
categories: ["AI", "Developer Tools"]
tags: ["kiro", "ai", "developer-tools", "productivity"]
image: "/images/blog/kiro-specs-hooks-cover.png"
ai_features:
  mermaid_diagrams: false
  code_snippets: true
  technical_depth: "intermediate"
content_type: "technical"
auto_toc: true
reading_time: true
draft: false
---

# Specs dan Agent Hooks di Kiro IDE: Cara Baru Kerja Bareng AI

Kalau kamu sudah coba Kiro IDE, mungkin kamu langsung fokus ke fitur chat-nya. Wajar. Tapi ada dua fitur yang justru bikin Kiro berbeda dari AI coding tools lain: **Specs** dan **Agent Hooks**.

## Masalah dengan AI Coding Tools Biasa

Skenario yang familiar: kamu minta AI untuk "buatkan fitur notifikasi". AI langsung nulis kode, tapi hasilnya tidak sesuai arsitektur, lupa handle edge case, tidak konsisten dengan design system.

Masalahnya bukan AI-nya bodoh. Masalahnya adalah tidak ada kontrak yang jelas antara kamu dan AI tentang apa yang harus dibangun, bagaimana caranya, dan standar apa yang harus dipenuhi.

Specs dan hooks adalah jawaban Kiro untuk masalah ini.

---

## Specs: Kontrak Tertulis antara Kamu dan AI

Spec adalah dokumen terstruktur yang mendefinisikan sebuah fitur sebelum satu baris kode pun ditulis. Strukturnya tiga file:

```
.kiro/specs/nama-fitur/
├── requirements.md  # Apa yang harus dibangun
├── design.md        # Bagaimana arsitekturnya
└── tasks.md         # Checklist implementasi
```

### requirements.md — Kontrak Fungsional

Tulis apa yang harus bisa dilakukan fitur ini dalam bahasa kebutuhan, bukan teknis:

```markdown
## R1 — Real-time Notifications

- R1.1: Notifikasi muncul tanpa page refresh via SSE
- R1.2: Badge count terupdate otomatis
- R1.3: Toast popup untuk notifikasi penting
```

Setiap requirement punya ID (R1.1, R1.2) supaya bisa direferensikan di tasks. AI agent tahu persis requirement mana yang sedang dikerjakan.

### design.md — Kontrak Teknis

Definisikan arsitektur: file apa yang dibuat, data flow-nya bagaimana, interface TypeScript-nya seperti apa. AI membaca ini sebelum mulai coding — hasilnya langsung sesuai arsitektur yang kamu inginkan.

### tasks.md — Kontrak Eksekusi

Ini yang paling powerful. Tasks adalah checklist yang dikerjakan agent satu per satu:

```markdown
## Task 1 — SSE Endpoint

- [ ] Buat src/routes/api/notifications/stream/+server.ts
- [ ] Implementasi ReadableStream dengan heartbeat 30 detik
- [ ] Auth guard: redirect jika tidak login
- [ ] Jalankan pnpm run check
```

Kamu bilang "kerjakan Task 1". Agent tahu persis scope-nya. Selesai, kamu review, baru lanjut Task 2.

Tanpa spec, AI bekerja seperti kontraktor tanpa blueprint. Dengan spec, kamu punya kontrol granular, konteks persisten, dan acceptance criteria yang jelas.

---

## Agent Hooks: Guardrails Otomatis

Hooks adalah automation yang jalan sendiri saat event tertentu terjadi. Formatnya file `.kiro.hook`:

```json
{
  "enabled": true,
  "name": "Type Check on Save",
  "version": "1",
  "when": {
    "type": "fileEdited",
    "patterns": ["src/**/*.svelte", "src/**/*.ts"]
  },
  "then": {
    "type": "runCommand",
    "command": "pnpm run check"
  }
}
```

Setiap kali file `.svelte` disimpan, `pnpm run check` jalan otomatis.

### Dua Jenis Action

**runCommand** — jalankan shell command langsung.

**askAgent** — minta AI agent melakukan sesuatu, misalnya verifikasi keamanan:

```json
{
  "enabled": true,
  "name": "Payment Security Guard",
  "version": "1",
  "when": {
    "type": "fileEdited",
    "patterns": ["src/routes/api/payments/**"]
  },
  "then": {
    "type": "askAgent",
    "prompt": "File payment diedit. Verifikasi signature verification masih intact dan tidak ada security check yang dihapus."
  }
}
```

### Event Triggers

| Event             | Kapan Jalan               |
| ----------------- | ------------------------- |
| fileEdited        | Saat file disimpan        |
| fileCreated       | Saat file baru dibuat     |
| postTaskExecution | Setelah spec task selesai |
| preToolUse        | Sebelum agent pakai tool  |
| userTriggered     | Manual trigger            |

### Hooks sebagai Institutional Knowledge

Ini yang sering terlewat: hooks bukan hanya automation, tapi cara mengkodifikasi aturan tim ke dalam sistem. Aturan yang biasanya hidup di dokumen yang jarang dibaca, sekarang di-enforce otomatis.

---

## Workflow Lengkap

1. Tulis spec (requirements → design → tasks)
2. Minta agent kerjakan Task 1
3. Hook postTaskExecution → auto pnpm run check
4. Hook fileEdited → verifikasi compliance
5. Review hasil, approve atau minta revisi
6. Lanjut Task 2

Kamu tetap yang memimpin. AI yang mengeksekusi. Hooks yang menjaga kualitas.

---

## Coba Mulai dari Sini

Coba mulai dari satu spec kecil dan dua atau tiga hooks yang paling relevan dengan pain point tim kamu. Rasakan bedanya ketika AI tidak lagi mengarang sendiri arsitektur dan aturan, tapi membaca blueprint yang kamu kasih.

Karena yang lebih menarik lagi, Kiro punya dua wajah: IDE dan CLI. Kapan pakai yang mana? Yuk, kita bedah di artikel berikutnya.

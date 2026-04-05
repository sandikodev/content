---
title: "Agent Steering dan Skills di Kiro: Mengarahkan AI agar Sesuai Keinginan"
description: "Specs dan hooks sudah, sekarang belajar mengarahkan AI agent melalui steering rules dan skills agar bekerja sesuai standar tim."
date: 2026-04-01T12:00:00+07:00
author: "Sandikodev"
categories: ["AI", "Developer Tools"]
tags: ["kiro", "steering", "skills", "ai", "workflow"]
image: "/images/blog/kiro-steering-skills-cover.png"
ai_features:
  mermaid_diagrams: false
  code_snippets: true
  technical_depth: "intermediate"
content_type: "technical"
auto_toc: true
reading_time: true
draft: false
---

# Agent Steering dan Skills di Kiro: Mengarahkan AI agar Sesuai Keinginan

Di artikel sebelumnya, kita sudah belajar tentang Specs (blueprint) dan Hooks (automation). Sekarang, kita akan belajar bagaimana **mengarahkan perilaku AI agent itu sendiri** melalui Agent Steering dan Skills.

Bayangkan kalau specs adalah "apa yang harus dibangun", dan hooks adalah "apa yang harus dilakukan saat event tertentu". Steering dan skills adalah "bagaimana AI harus berpikir dan bertindak".

---

## Masalah: AI Tidak Selalu Mengikuti Aturan

Meskipun sudah pakai specs dan hooks, tetap ada kemungkinan AI agent:

- Menggunakan style coding yang tidak konsisten
- Tidak mengikuti konvensi tim
- Melupakan preferensi keamanan
- Mengabaikan design tokens

Solusinya: **Steering Rules** dan **Skills** — dua mekanisme untuk "menanamkan" preferensi ke dalam DNA AI.

---

## Agent Steering: Aturan yang "Ditanam" ke AI

Steering rules adalah instruksi yang selalu dibaca AI sebelum mulai bekerja. Bedanya dari specs:

- **Specs**: untuk satu project/fitur spesifik
- **Steering**: berlaku global, untuk semua interaksi

### Cara Kerja

Ada beberapa jenis steering:

### 1. System Instructions

Instruksi level system yang berlaku untuk semua sesi. Contoh di `AGENTS.md`:

```markdown
# Agent Instructions

## Code Style

- Always use TypeScript strict mode
- Prefer functional components over class components
- Use design tokens from src/config/design-system.json

## Security

- Never log secrets or API keys
- Always validate input on server-side
- Use parameterized queries for database
```

Setiap kali AI membaca file ini, ia akan mengikuti aturan tersebut.

### 2. Inline Steering

Instruksi spesifik yang kamu kasih langsung di chat:

```
User: "Buat komponen button baru. Ingat: gunakan design system yang sudah ada,
bukan buat style baru. Cek existing Button.tsx dulu."
```

### 3. Directory-Level Steering

Aturan spesifik per direktori:

```
.kiro/steering/
├── frontend.md      # Aturan untuk /src/frontend/
├── backend.md       # Aturan untuk /src/backend/
└── database.md      # Aturan untuk /src/db/
```

Saat AI bekerja di `/src/frontend/`, ia akan baca `frontend.md` terlebih dahulu.

### Contoh Steering Nyata

Misalnya tim kamu punya aturan:

```markdown
# .kiro/steering/frontend.md

## Styling Rules

- SELALU gunakan Tailwind CSS, jangan inline styles
- Untuk spacing, gunakan scale dari design-system.json
- Untuk colors, gunakan CSS variables dari theme.json
- Hindari hardcoded hex colors

## Component Patterns

- Semua komponen harus pakai $props() (Svelte 5)
- Gunakan PascalCase untuk component names
- Export types di separate file

## Imports

- Urutan: built-in → external → internal
- Gunakan alias @/ untuk import dari src/
```

Dengan steering ini, setiap kali AI buat komponen di frontend, ia akan: -，自动 menggunakan Tailwind -、Mengambil colors dari CSS variables -、Menggunakan $props() untuk props -、Menggunakan alias @/

---

## Skills: Kemampuan Khusus untuk Task Tertentu

Skills adalah kumpulan instruksi yang mendefinisikan "kemampuan" spesifik untuk task tertentu. Bedanya dari steering:

- **Steering**: general rules untuk semua task
- **Skills**: definisi skill spesifik yang bisa dipanggil

### Format Skill

Skill didefinisikan dalam file `.md` di direktori `.kiro/skills/`:

```
.kiro/skills/
├── code-review.md
├── write-tests.md
├── refactor.md
└── security-audit.md
```

### Contoh: Code Review Skill

````markdown
# Skill: Code Review

## Description

Perform comprehensive code review following team standards.

## When to Use

- Ketika diminta "review this code"
- Ketika ada pull request baru
- Ketika ada merge request

## Review Criteria

### 1. Code Quality

- Apakah ada code smells?
- Apakah ada opportunities untuk extraction?
- Apakah complexity masih reasonable?

### 2. Security

- Apakah ada potential vulnerabilities?
- Apakah secrets sudah di-hardcode?
- Apakah input validation cukup?

### 3. Performance

- Apakah ada N+1 queries?
- Apakah ada unnecessary re-renders?
- Apakah ada memory leaks?

### 4. Testing

- Apakah test coverage cukup?
- Apakah edge cases sudah di-cover?
- Apakah test readable?

## Output Format

Setelah review, selalu output dalam format:

```markdown
## Review Summary

[Overall assessment]

## Issues Found

| Severity | Location | Issue | Suggestion |
| -------- | -------- | ----- | ---------- |
| High     | src/...  | ...   | ...        |
```
````

## Actions

- Jika ditemukan issues, jelaskan dengan detail
- Jika tidak ada issues, kasih konfirmasi
- Selalu propose solutions, bukan hanya kritik

```

### Cara Menggunakan Skill

Di chat, kamu bisa langsung panggil skill:

```

User: "Bisa lakuin code review untuk src/components/Button.tsx?"

```

AI akan load skill `code-review.md`, lalu perform review sesuai kriteria di dalamnya.

---

## Combo: Steering + Skills + Specs + Hooks

Sekarang kita punya empat pilar workflow dengan Kiro:

| Fitur | Fungsi | Scope |
|-------|--------|-------|
| **Specs** | Blueprint untuk satu fitur | Per project |
| **Hooks** | Automation saat event | Global |
| **Steering** | Aturan berpikir AI | Global + per directory |
| **Skills** | Kemampuan khusus untuk task | Per task type |

### Workflow Lengkap

Misalnya mau buat fitur baru:

```

1. Spec (requirements.md + design.md + tasks.md)
   → Defines WHAT to build

2. Steering (.kiro/steering/frontend.md)
   → Defines HOW to write code

3. Skills (.kiro/skills/write-tests.md)
   → Defines HOW to test

4. Hooks (post-task-execution.kiro.hook)
   → Defines WHAT HAPPENS AFTER

````

AI tidak hanya diberi tahu "apa yang harus dibangun", tapi juga:
- "Bagaimana menulis kodenya" (steering)
- "Bagaimana mengetesnya" (skill)
- "Apa yang dilakukan setelahnya" (hook)

---

## Contoh Praktis: Feature Migration Redux → Svelte Store

Mari combine semuanya:

### Step 1: Buat Spec

```markdown
# .kiro/specs/redux-to-svelte/

## Requirements
- R1: Semua Redux store di-migrate ke Svelte stores
- R2: Tidak ada breaking changes di API
- R3: Unit tests tetap jalan setelah migration

## Design
- Store location: src/lib/stores/
- Naming: camelCase, ending with $store
- Migration strategy: one store at a time

## Tasks
- [ ] Inventory semua Redux stores
- [ ] Migrasi store pertama sebagai pilot
- [ ] Update semua consumers
- [ ] Jalankan tests
- [ ] Review code
````

### Step 2: Steering

```markdown
# .kiro/steering/stores.md

## Migration Rules

- Gunakan Svelte 5 runes ($state, $derived)
- Untuk stores dengan async, gunakan $effect
- Export store sebagai function yang return { state, actions }
- Jaga backward compatibility dengan export alias
```

### Step 3: Skill

```markdown
# .kiro/skills/migration.md

## Migration Pattern

- Always create new file, never overwrite
- Keep old file for reference during migration
- Add TODO comments for items that need manual attention
- Update import statements in all consumers
```

### Step 4: Hook

```json
{
  "enabled": true,
  "name": "Migration Tests",
  "version": "1",
  "when": {
    "type": "fileCreated",
    "patterns": ["src/lib/stores/*.ts"]
  },
  "then": {
    "type": "runCommand",
    "command": "pnpm run test:stores"
  }
}
```

---

## Best Practices

### 1. Mulai dari yang kecil

Jangan langsung buat 20 steering rules. Mulai dari 3-5 aturan yang paling sering dilanggar.

### 2. Review dan iterasi

Steering rules bukan "set and forget". Setiap kali AI membuat mistake, update steering untuk prevent terulang.

### 3. Dokumentasikan

Steering rules yang tidak didokumentasikan = tidak dibaca AI. Pastikan setiap rule punya context why-nya.

### 4. Gunakan version control

Steering rules di-commit ke repo. History steering rules bisa lihat bagaimana standar tim berevolusi.

---

## Kesimpulan

Agent Steering dan Skills adalah layer terakhir untuk mengontrol AI agent. Dengan empat pilar ini:

- **Specs** → Apa yang harus dibangun
- **Hooks** → Apa yang terjadi setelahnya
- **Steering** → Bagaimana AI harus berpikir
- **Skills** → Kemampuan khusus yang dimiliki

Kamu punya kontrol penuh atas AI agent. Bukan lagi "kerjakan ini", tapi "kerjakan dengan cara ini, menggunakan skill ini, dan setelah selesai lakukan itu".

Ini adalah shift dari "AI sebagai tool" ke "AI sebagai team member yang bisa diasah sesuai standar tim".

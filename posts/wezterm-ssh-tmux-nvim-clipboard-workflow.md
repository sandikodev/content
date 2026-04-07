---
title: "Clipboard Seamless: WezTerm + SSH + tmux + Neovim"
description: "Copy-paste antara Neovim di remote server dan laptop lokal tanpa plugin tambahan, tanpa X forwarding, hanya dengan OSC 52 dan konfigurasi minimal."
date: 2026-04-07T10:00:00+07:00
author: "Sandikodev"
categories: ["Tools"]
tags:
  [
    "wezterm",
    "tmux",
    "neovim",
    "vim",
    "ssh",
    "terminal",
    "linux",
    "productivity",
    "workflow",
    "clipboard",
  ]
image: "/images/blog/wezterm-ssh-tmux-nvim-clipboard-visual.png"
ai_features:
  mermaid_diagrams: false
  code_snippets: true
  technical_depth: "intermediate"
content_type: "technical"
auto_toc: true
reading_time: true
draft: false
---

Satu hal yang selalu mengganggu saat kerja via SSH: copy-paste yang tidak nyambung. Kamu select teks di Neovim, tekan `y`, tapi clipboard laptop kamu tetap kosong.

Artikel ini mendokumentasikan setup yang saya gunakan sehari-hari — dan setelah konfigurasi ini, `y` di Neovim langsung masuk ke clipboard laptop.

---

## Stack

```
Laptop (WezTerm) → SSH → Server Linux (tmux → Neovim)
```

Tidak ada port forwarding. Tidak ada X server. Hanya SSH biasa.

---

## Kenapa Bisa Bekerja: OSC 52

**OSC 52** adalah escape sequence standar yang memungkinkan program di remote mengirim teks langsung ke clipboard lokal melalui koneksi SSH yang sudah ada. WezTerm support ini secara native — kita hanya perlu mengaktifkannya di tmux dan Neovim.

---

## Konfigurasi

### tmux (`~/.tmux.conf`)

```bash
# Clipboard via OSC 52
set -g set-clipboard on
set -g allow-passthrough on
set -ag terminal-overrides ',*:Ms=\E]52;c;%p2%s\007'

# Copy mode dengan vim keys
setw -g mode-keys vi
bind -T copy-mode-vi v send -X begin-selection
bind -T copy-mode-vi y send -X copy-selection-and-cancel
```

Reload tanpa restart:

```bash
tmux source-file ~/.tmux.conf
```

### Neovim (`~/.config/nvim/init.lua`)

```lua
vim.opt.clipboard = "unnamedplus"
```

Neovim 0.10+ sudah built-in OSC 52 — satu baris ini sudah cukup.

### Vim biasa (`~/.vimrc`)

Untuk Vim, tambahkan plugin [vim-oscyank](https://github.com/ojroques/vim-oscyank):

```vim
set clipboard=unnamedplus
Plug 'ojroques/vim-oscyank'
autocmd TextYankPost * if v:event.operator is 'y' && v:event.regname is '' | execute 'OSCYankRegister "' | endif
```

---

## Copy-Paste dalam Praktik

### Dengan Mouse

WezTerm menampilkan context menu saat klik kanan — lengkap dengan Cut, Copy, Paste. Ini bekerja bahkan saat Neovim sedang dalam VISUAL mode di dalam tmux di dalam SSH.

![Neovim VISUAL mode dengan teks terseleksi, siap di-copy via klik kanan](/images/blog/wezterm-ssh-tmux-nvim-clipboard-visual.png)

![Context menu WezTerm: Cut, Copy, Paste tersedia langsung](/images/blog/wezterm-ssh-tmux-nvim-clipboard-menu.png)

### Dengan Keyboard di Neovim

| Key | Aksi |
|-----|------|
| `v` | Visual mode (per karakter) |
| `V` | Visual line (per baris) |
| `Ctrl+v` | Visual block (kolom) |
| `viw` | Select satu kata |
| `vi"` | Select isi dalam tanda kutip |
| `vip` | Select satu paragraf |
| `y` | Yank → langsung ke clipboard laptop |

### Dengan Keyboard di tmux (luar Neovim)

```
Prefix + [    → masuk copy mode
v             → mulai select
y             → copy & keluar
```

---

## Perbandingan Metode Clipboard via SSH

| Metode | Kelebihan | Kekurangan |
|--------|-----------|------------|
| **OSC 52** | Native, zero dependency | Butuh terminal yang support |
| X11 Forwarding | Lengkap | Lambat, butuh X server di laptop |
| `xclip` / `xsel` | Familiar | Tidak bekerja tanpa X forwarding |
| tmux-yank plugin | Mudah setup | Butuh `xclip`/`xsel` di server |

OSC 52 adalah pilihan terbaik untuk setup SSH murni — tidak ada overhead, tidak ada dependency tambahan di server.

---

## Verifikasi OSC 52 Bekerja

Jalankan ini di dalam tmux via SSH, lalu coba paste di laptop:

```bash
printf '\033]52;c;%s\007' "$(echo -n 'test osc52' | base64)"
```

Kalau berhasil paste "test osc52" di laptop, setup sudah benar.

---

## Kesimpulan

Tiga baris config di tmux dan satu baris di Neovim — clipboard antara laptop dan remote server langsung seamless. Tidak perlu plugin berat, tidak perlu port forwarding, tidak perlu X server.

Setup ini juga kompatibel dengan Vim biasa menggunakan plugin vim-oscyank, sehingga bisa dipakai di server yang belum punya Neovim.

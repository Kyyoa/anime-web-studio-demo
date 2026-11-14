# 📌 PROGRESS — anime-web-studio & Demo KISEKI

> **Handoff dokumen ini ke session baru.** Kalau kamu agent yang baru mulai:
> baca file ini sampai habis, lalu ikuti section "TUGAS BERIKUTNYA".
> Terakhir diupdate: 2026-08-31 (session sebelum user tidur).

---

## ✅ STATUS: SUDAH SELESAI (jangan diulang)

### 1. Skill `anime-web-studio` v1.1.0 — LIVE & TERUJI

- **Canonical:** `~/.hermes/skills/web-development/anime-web-studio/SKILL.md` (445 baris)
- **Symlink OpenCode:** `~/.config/opencode/skills/anime-web-studio` → canonical (verified resolve)
- **Fitur:**
  - 7 art direction + font pairing Jepang (RocknRoll One, Zen Kaku Gothic New, DotGothic16, dll)
  - Anime Signature Elements: 7 CSS recipes (speed lines, halftone, diagonal cuts, vertical katakana, giant kanji, bold outlines, SFX onomatopoeia)
  - **Registry 3-TIER:**
    - 🟢 TIER 1 production-safe: itch.io anime assets (verifikasi per-aset), SpriteCook, KayKit, Papyszoo CC0, Japan Map SVG, Khronos (CC0 only), generated art
    - 🟡 TIER 2 demo-only: Waifu.pics, Waifu.im (manifest wajib `license:"unknown"`, `commercial_use:false`)
    - 🔴 TIER 3 dilarang: GI-Assets (rip Genshin, GPL repo ≠ lisensi aset), output AssetStudio/FModel, screenshot anime komersial
  - License Gate risk-tiered (bukan hard-block) + Mode Check (production vs demo) di workflow
  - Tool integrations: Firecrawl, Camofox, **Pollinations.ai** (gratis tanpa key), Higgsfield, here.now
  - Rationalization table + DO NOT (termasuk "repo GPL ≠ aset bebas", "tool legal ≠ output legal")
- **Verifikasi:** subagent test 4/4 skenario klasifikasi tier benar (GI-Assets ditolak, waifu.pics demo boleh, production→Tier 1, generate→Pollinations)

### 2. Demo KISEKI — `~/Projects/anime-web-studio-demo/`

- `index.html` (~675 baris): art direction **Modern Streaming** (`#0d0d12` bg / `#ff4757` accent, 60-30-10), RocknRoll One + Zen Kaku Gothic New, semua signature elements, content vocabulary anime (episode rail, character profile JP+romaji+CV+quote, rating badge, season badge, countdown, news)
- `assets/img/` — 10 gambar AI-generated Pollinations.ai (348KB total): `hero-b.jpg`, `char-rin.jpg`, `char-kenji.jpg`, `char-guardian.jpg`, `ep1..6.jpg` (640×400)
- `assets/asset-manifest.json` — valid, 7 entri, semua `generated/pollinations.ai`, `license:"user-defined"`
- **MASALAH AKTIF:** hasil generate terlihat **donghua** (semi-realistis painterly), user kurang suka → lihat tugas berikutnya

### 3. Localhost

```
http://localhost:8742  (juga via LAN: http://192.168.100.7:8742)
```
Restart kalau mati:
```bash
cd ~/Projects/anime-web-studio-demo && setsid nohup python3 -m http.server 8742 --bind 0.0.0.0 > /tmp/opencode/anime-demo-server.log 2>&1 < /dev/null &
```

---

## 🎯 TUGAS BERIKUTNYA: Style-Lock Anime (SUDAH DISETUJUI USER, BELUM DIEKSEKUSI)

**User feedback:** "ini lebih bagus tapi belum seperti anime, ini lebih kaya donghua animasi china, aku kurang suka"

**Keputusan user (TERKUNCI, jangan tanya lagi):**
1. Regenerate **SEMUA 10 aset dari nol** (bukan keep favorit)
2. Post-process **cel-flat + lineart tipis** (test lineart di 1 aset dulu; kalau over-darken → flat cel saja)

### Step 1 — Regenerate 10 aset (Pollinations, prompt anti-donghua)

Base prompt WAJIB di semua slot:
```
flat 2D anime style, Japanese anime screenshot, 1990s anime film look,
hand-drawn animation, simple flat colors, two-tone shading, big expressive
anime eyes, clean lineart, minimal gradients, no photorealism, no 3D render,
no CGI look, no glossy skin, no painterly, no semi-realistic
```

| Slot | File | Ukuran | Subjek prompt |
|------|------|--------|---------------|
| Hero A | hero-a.jpg | 880×1100 | girl short dark hair red scarf, night neon city, dynamic |
| Hero B | hero-b.jpg | 880×1100 | variasi front view |
| Karakter 1 | char-rin.jpg | 700×900 | serious teen girl, red scarf, city bokeh, red accent |
| Karakter 2 | char-kenji.jpg | 700×900 | friendly young man glasses, blue jacket, blue accent |
| Karakter 3 | char-guardian.jpg | 700×900 | hooded figure golden lantern, garden, gold accent |
| Episode 1-6 | ep1..6.jpg | 640×400 | railway dusk / alley lamp 2am / festival fireworks / rooftop garden rain / music box purple / pink sunset bridge silhouettes |

**Pola curl (sequential, JANGAN parallel — rate limit 1 request/IP):**
```bash
curl -s -m 90 -o out.jpg "https://image.pollinations.ai/prompt/<URL-ENCODED>?width=W&height=H&nologo=true&model=sana&seed=N"
```
- Kalau dapat JSON error ("Too Many Requests"/"Queue full") → sleep 10, retry (max 5x)
- Pilih hero terbaik dari A/B (lihat gambarnya, jangan asal)

### Step 2 — Cel-flat post-process (ImageMagick — SUDAH TERUJI)

```bash
# flat cel bands (hasil bersih, terverifikasi):
magick <src> -resize <W>x<H>^ -gravity center -extent <W>x<H> -dither none -colors 8 -colorspace sRGB -quality 88 <out>
```
- Lineart tipis (BELUM BERHASIL BERSIH — percobaan awal over-darken):
  percobaan lama: `\( +clone -colorspace gray -edge 1 -negate -level 0%,70% \) -compose multiply` → terlalu gelap.
  Coba yang lebih ringan: edge hanya di luminance tinggi, atau `-edge 0.5`, threshold lebih selektif. Test di 1 aset dulu; kalau tetap jelek → flat cel saja.
- Simpan original generate di `assets/img/src/` (bukti manifest)
- Tuning kalau masih "donghua": `-colors 6` (lebih flat)

### Step 3 — Integrate + QA

1. Nama file output SAMA dengan sekarang → HTML tidak perlu diubah (cuma replace file)
2. Visual QA: buka tab Camofox fresh:
   ```bash
   TAB=$(curl -s -X POST http://localhost:9377/tabs -H 'Content-Type: application/json' -d '{"url":"http://localhost:8742/index.html","userId":"opencode-demo","sessionKey":"anime-qa-N"}' | python3 -c "import sys,json;print(json.load(sys.stdin)['tabId'])")
   # screenshot via snapshot:
   curl -s "http://localhost:9377/tabs/$TAB/snapshot?includeScreenshot=true&userId=opencode-demo" → JSON field screenshot.data (base64 PNG)
   # scroll: POST /tabs/$TAB/scroll {"userId":"opencode-demo","direction":"down","amount":1100}
   # close: DELETE /tabs/$TAB?userId=opencode-demo
   ```
3. 3-second test per section (hero / episode rail / karakter): "orang awam langsung recognize anime Jepang?"
   Kriteria: hard-edged flat shading (bukan gradient lembut), wajah stylized rounder, aset kohesif satu gaya
4. Update `asset-manifest.json`: tambah `"post_process": "ImageMagick cel-flat (-dither none -colors 8)"` di entri
5. Refresh `http://localhost:8742` → minta user review

### Kriteria sukses
1. Shading = flat hard-edged bands ✅ bukan painterly
2. Wajah = anime stylized (round eyes) bukan realistis
3. Semua aset satu gaya kohesif
4. User refresh localhost → bilang "ini anime" 😄

---

## 🧠 LEARNINGS (penting, jangan hilang)

1. **Model Pollinations (sana/flux/turbo) semuanya bias painterly/semi-realistis** → keyword `cel shading/flat color` saja TIDAK cukup. SOLUSI: post-process ImageMagick wajib.
2. `-dither none -colors 8` = flat cel BERSIH ✅ | `-posterize` = dithering noise ❌
3. Preferensi user: wajah **rounder anime-style** (dulu suka hero-b & char-rin karena wajahnya), ANTI wajah realistis = donghua
4. Camofox: tab ID bisa stale ("browser_restarted") → selalu **buat tab fresh** per sesi QA; endpoint butuh `userId` + `sessionKey`
5. Pollinations rate limit: 1 concurrent/IP → error JSON → retry dengan sleep
6. Pollinations juga bisa via `model=flux` / `model=turbo` tapi sama-sama painterly (sudah dites) → tetap pakai sana + post-process
7. Environment: **no GPU** (Intel UHD) → ComfyUI lokal tidak viable; Higgsfield butuh auth (user belum punya akun)
8. Convert vs magick: pakai `magick` (IMv7)

---

## 📋 OPSIONAL (belum dikomit user — tanya nanti)

- Deploy demo ke here.now (public URL)
- Coba art direction lain (Manga / Kawaii / Cyber) sebagai variasi
- "Pemantapan skill" — user bilang mau lanjut besok untuk ini (kemungkinan: tambahkan learning cel-flat pipeline ke SKILL.md sebagai bagian Hero Art strategy — akurat setelah divalidasi di demo)

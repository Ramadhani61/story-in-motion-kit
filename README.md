# Story in Motion Kit — pipeline video "30 Hari Juna"

Flat 2D, 1920×1080, 30 fps. Semua gambar dibuat dengan kode (SVG → PNG → PIL), suara efek disintesis (tanpa unduhan), musik latar opsional.

## Alur satu episode
```bash
KIT=/path/ke/story-in-motion-kit
bash $KIT/tools/setup.sh                       # sekali per sesi: pip, font, model Whisper (GitHub)
mkdir -p ep && cd ep && mkdir audio
# 1. taruh naskah.md (format: '## BAGIAN 1 …', baris '[ADEGAN…]' = catatan adegan, '---' pemisah)
python3 $KIT/tools/split_script.py naskah.md   # -> narasi_bagian{n}.txt + narasi_flat{n}.txt
# 2. taruh audio/p1.mp3 p2.mp3 p3.mp3 (+ audio/bgm.mp3 kalau pakai musik)
python3 $KIT/tools/asr_align.py                # -> anchors.json (waktu tiap kalimat), env.npy (lip-sync)
python3 $KIT/tools/build_assets.py             # -> assets/ (latar + sprite)
# 3. tulis story.py (lihat template/story_example_krl.py)
python3 $KIT/tools/render.py still 12 80 300   # cek frame dulu -> still_*.jpg
python3 $KIT/tools/render.py all 2             # render semua adegan + mix audio -> final.mp4
```
Ubah satu adegan: hapus `scenes/NN.mp4`, jalankan `render.py all` lagi (adegan lain dipakai ulang).

## Menulis story.py
`from scenes import *` memberi semua helper. Waktu memakai **anchor kalimat**: `S(bagian, indeks_kalimat)` = detik global saat kalimat itu mulai diucapkan (indeks dari 0, urutan kalimat di narasi_flat). `G['offsets']['2']` = awal bagian 2, `END_VO`, `TOTAL` (= END_VO + 4 detik end card).

```python
@scene(S(1, 21) - 0.05, 'peron_serpong_subuh', day=1, jam='05.40', cal=True, pan=0)
def s06(fr, t, T):            # fr = frame PIL RGB, t = detik dalam adegan, T = detik global
    juna(fr, T, 960, 1075, 'kaget', f=0, s=0.85)
    chip(fr, T, S(1, 22) + 0.2, 'teks chip', 1400, 380, MUST, INK, 44)
cue(S(1, 24), 'train', -4)    # efek suara (nama, dB)
calcue(S(1, 21) - 0.05)       # SFX standar transisi kalender
```
- `scene(start, bg, day=None, jam=None, cal=False, pan=0, hud_on=True)`: adegan berakhir saat adegan berikutnya mulai. `cal=True` memunculkan transisi kalender "HARI KE-x" 1,8 detik. `day`/`jam` menampilkan HUD.
- **Juna**: `juna(fr,T,x,y,expr,f,pose='diri'|'angkat',s,speak=True,flip,sweat)`. expr: netral senang kaget cemas mikir (+ lelah f=2, habis f=3). f = level lelah 0–3. Mulut bergerak otomatis mengikuti suara (`speak=False` saat tokoh lain berbicara). `bigjuna(fr,T,x,y,expr,f,s)` untuk close-up (habis/3, lelah/2, senang/0, netral/1, kaget/0, cemas/1).
- **Tokoh lain**: `person_(fr,T,name,expr,x,y,s,speak,flip)`. name: om bayu darto sri bos turis warga rina bapak k0..k9. expr: netral senang bicara cemas kesal sedih kaget. `crowd(fr,T,[(y,scale,n,x0,x1),…],mood,seed)` untuk kerumunan.
- **Teks/grafis**: chip, bubble(…, tail='bl'|'br', t1=selesai), stamp_text, title_text(fr,t,t0_lokal,…), hand_text (tulisan tangan Caveat, efek mengetik), notif, phone, card, counter_box, scale_beam, confetti, arrow, gold_bar, bills, nb_page (buku catatan), train(fr,x,y,'train'|'train_long').
- **Latar** (assets/bg_*.png): kos_pagi kos_pagi_akhir kos_malam kamar_subuh meja rumah_om rumah_om_malam kantor kantor_sore toko_emas toko_elek pasar gudang spbu jalan lorong warung bali bank kiriman peron_{serpong_subuh,serpong_pagi,serpong_sunrise,tanahabang_pagi,tanahabang_sore,manggarai_sore} gerbong_pagi gerbong_subuh tangga tangga_sore dapur cermin plain_must plain_teal plain_navy sinar_navy sinar_krem. Latar baru: buat `story_assets.py` berisi `EXTRA_BG = {'nama': svg_string}` (pakai helper di core/bg.py & bg2.py: wall_floor, window, clock, rect, text, peron(nama,'pagi'|'sore'|'subuh'|'malam'), plain(...)), lalu jalankan build_assets.py lagi.
- **SFX**: whoosh whoosh2 pop pop_hi pop_lo ding buzz kaching coin scribble pageflip stamp drop riser sparkle sad party tick crowd engine crickets bubbling typing wah alarm train door announce steps tv drip snore.

## Aturan brand
Pembuka "Oke. Kita catat dulu." · penutup "Aku Juna, ini Story in Motion. Sampai ketemu di catatan berikutnya." · buku catatan merah, HUD HARI x/30, kalender dicoret, level lelah. Jangan pakai nama Laras/Dika. Selalu cek still frame sebelum render penuh (teks bertumpuk, tokoh tertutup).

# Clarion

O'zbekcha nutqni matnga aylantiruvchi Claude Code plugini. Audio yoki
videoni matnga, vaqtli segmentlarga yoki SRT/VTT subtitrga o'giradi.
Ichida [UzASR](https://api.uzasr.uz) API ishlaydi: o'zbek tiliga
moslab o'qitilgan GigaAM modeli va KenLM.

Plugin ikki qismdan iborat:

- **MCP server** `https://api.uzasr.uz/mcp`. API kalit o'rnatishda bir
  marta so'raladi va OS keychain'ida saqlanadi.
- **`/clarion:transcribe` skill'i.** Fayl hajmini tekshiradi, kerak
  bo'lsa ffmpeg bilan siqadi, yuklaydi, transkript qiladi va natijani
  faylga yozadi.

## O'rnatish

Claude Code ichida:

```
/plugin marketplace add rustammdev/clarion
/plugin install clarion@clarion
```

O'rnatishda UzASR API kaliti (`uzasr_live_…`) so'raladi. Kalitni keyin
almashtirish: `/plugin configure clarion@clarion`. Plugin uchun alohida
kalit oching, shunda uni boshqa integratsiyalarga tegmasdan bekor qilish
mumkin bo'ladi.

## Ishlatish

Oddiy so'z bilan aytish kifoya, skill o'zi ishga tushadi:

- "dars.mp4 ni o'zbekcha subtitrga aylantirib dars.srt ga yoz"
- "intervyu.mp3 ni matnga o'gir va asosiy fikrlarni qisqacha yoz"
- "podcasts/ papkasidagi hamma mp3 larni transkript qil"

Yoki to'g'ridan-to'g'ri: `/clarion:transcribe lesson.mp4 srt`.

Formatlar: `text`, `segments` (vaqtli qatorlar), `srt`, `vtt`. Modellar:
`uzasr-large` (asosiy, eng aniq) va `uzasr-base` (tezroq, arzonroq).

## Cheklovlar

Fayl 25 MB gacha, audio 3 soatgacha. Kalitda transcribe kutayotgan
upload'lar 10 tagacha. Yuklash va yuklab olish havolalari bir martalik va
15 daqiqa amal qiladi. Faqat o'zbek tili.

## Xavfsizlik va maxfiylik

- Skill ruxsatsiz faqat Clarion MCP tool'larini chaqira oladi. `curl` va
  `ffmpeg` odatdagidek ruxsat so'raydi.
- Audio serverda faqat ishlash uchun turadi va ishlangach o'chiriladi.
  Transkript 7 kundan keyin tozalanadi.
- `save_to` bilan natija to'g'ridan-to'g'ri faylga tushadi va agent
  kontekstiga kirmaydi.

## Ishlab chiqish

```
claude plugin validate . --strict
claude --plugin-dir .
```

## Litsenziya

[MIT](LICENSE)

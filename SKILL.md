---
name: zimage-prompt-writing
description: "Use when writing Z-Image-Turbo image prompts. PE method."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, darwin, linux]
metadata:
  hermes:
    tags: [z-image, image-generation, prompt-engineering, comfyui]
    related_skills: [comfyui-desktop-windows, ltx25-prompt-writing]
---

# Z-Image-Turbo Prompt Yazma (resmi PE metodolojisi)

## When to Use

- Z-Image-Turbo ile görsel üretirken prompt yazılacaksa (bu makinede ComfyUI
  `image_z_image_turbo_int8` workflow'u).
- Kullanıcı görsel fikri verdiğinde → PE prosedürüyle son prompt'u üret.

Kaynaklar (2026-08'de incelendi):
- Model kartı: `https://huggingface.co/Tongyi-MAI/Z-Image-Turbo`
- PROMPTING GUIDE (resmi takım cevapları): `.../discussions/8`
- Resmi Prompt Enhancer (PE) şablonu: Space `.../blob/main/pe.py`

## Model Gerçekleri (prompt'u şekillendiren farklar — LTX ile KARIŞTIRMA)

- **Negatif prompt YOK**: few-step distilled (Decoupled-DMD + DMDR/RL), CFG-free inference.
  Diffusers'ta `guidance_scale=0.0`, ComfyUI'da `cfg=1`. Negatif prompt yazmak zaman kaybıdır,
  model onu görmez. (Z-Image TEMEL modeli CFG + negatif prompt destekler; Turbo desteklemez.)
- **Uzun ve detaylı prompt en iyisi** — resmi tavsiye: "long and detailed prompts".
- **Dil: İngilizce tercih et** (topluluk doğrulaması: aynı sahne İngilizce PE çıktısıyla çok
  daha iyi). Çince render güçlüdür — render edilecek metin prompt'ta iki dilde de yazılabilir.
- **Metin render'ı GÜÇLÜ yan**: LTX'in tam tersi — görseldeki metni prompt'ta tırnak içinde
  aynen yaz (bkz. PE adım 4).
- **Token bütçesi**: demo 512 token; lokal 1024 (`max_sequence_length=1024`). ~0.75 kelime/token
  → 512 tok ≈ 380 kelime; 1024 tok ≈ 750 kelime. TE = Qwen3-4B (ComfyUI'da `lumina2` tipi).
- **Yüksek determinizm**: aynı prompt benzer render verir; çeşitlilik için prompt'a ANLAMLI
  yeni detay ekle (cfg ayarı işe yaramaz — zaten 0). Seed değişimi sınırlı varyasyon verir.
- **Adımlar**: 9 API adımı = 8 DiT forward (ComfyUI'da `steps=8`).
- **Mimari**: 6B S3-DiT (single-stream) — metin + görsel tokenlar tek akışta; metin
  kompozisyona doğrudan işler.
- İlham galerisi: `https://zimage.net/inspiration` (kopyalanabilir prompt'lar).

## Resmi PE (Prompt Enhancer) Metodolojisi — 4 adım

Benim görevim pe.py'deki rolü bizzat oynamak: "mantık kafesindeki vizyoner sanatçı" —
kullanıcı niyetine sadık, detay dolu, estetik, doğrudan T2I'ya girebilen ultimat görsel
betimleme üretmek.

1. **Çekirdek unsurları kilitle** — kullanıcı prompt'undaki DEĞİŞMEZ unsurlar: özne, sayı,
   eylem, durum + belirtilen IP isimleri, renkler, metinler. Bunlar mutlak korunur.
2. **Üretken akıl yürütme kararı** — istek doğrudan sahne değil de çözüm gerektiriyorsa
   ("X nedir?", "tasarla", "şu sorunun çözümünü göster"): önce zihninde TAM, somut,
   görselleştirilebilir bir plan kur; betimleme bunun üzerine inşa edilir.
3. **Profesyonel estetik + gerçekçilik enjeksiyonu** — beş eksen:
   kompozisyon · ışık/gölge atmosferi · malzeme/doku · renk şeması · katmanlı mekân derinliği.
4. **Metin elemanlarının kesin işlenmesi (KRİTİK)** —
   - Final görselde görünmesi istenen TÜM metni kelimesi kelimesine yaz, İngilizce çift
     tırnak içinde: `"..."` (açık üretim talimatı).
   - Poster/menü/UI gibi tasarım işlerinde: tüm metin + font + tipografi/düzen.
   - Sahnedeki tabela/ekran/yol işareti metinleri: içerik + konum, boyut, malzeme.
   - Kendi akıl yürütmende eklediğin metinler (grafik, çözüm adımları): aynı kurallar.
   - Metin yoksa → tüm enerji saf görsel detaya.

## Çıktı Kuralları (PE'nin katı şartları)

- Objektif ve somut. Metafor YASAK, duygusal retorik YASAK.
- `"8K"`, `"masterpiece"` gibi meta etiketler ve çizim talimatları YASAK.
- Sadece final prompt çıkar — açıklama, önsöz, alternatif YOK.

## Prompt İnşa Prosedürü (fikir → final prompt)

1. Fikri al → çekirdek unsurları kilitle (özne/sayı/eylem/renk/metin).
2. "Üretken akıl yürütme" gerekir mi? (tasarım/çözüm istekleri) → önce görselleştirilebilir plan.
3. 5 eksende estetik detay ekle (kompozisyon, ışık, doku, renk, derinlik).
4. Metin audit: görselde metin varsa tırnak içinde aynen yaz (+font/düzen gerekirse).
5. İngilizce yaz; token bütçesini tut (hedef ~100-380 kelime; uzun işler 750'ye kadar).
6. Negatif prompt YAZMA. Meta etiket kullanma.
7. ComfyUI parametreleri (bu makine): steps=8, cfg=1, 1024x1024, sampler res_multistep /
   scheduler simple, TE `qwen_3_4b_fp8_mixed` (lumina2), UNET `z_image_turbo_int8_convrot`.

## Örnek stil (README'den — resmi showcase prompt'u)

> Young Chinese woman in red Hanfu, intricate embroidery. Impeccable makeup, red floral
> forehead pattern. Elaborate high bun, golden phoenix headdress, red flowers, beads. Holds
> round folding fan with lady, trees, bird. Neon lightning-bolt lamp (⚡️), bright yellow glow,
> above extended left palm. Soft-lit outdoor night background, silhouetted tiered pagoda
> (西安大雁塔), blurred colorful distant lights.

Not: kısa cümleler, somut nesneler, ışık + arka plan detayı, Çince metin doğrudan prompt'ta.

## Destek dosyaları

- `references/pe-template.md` — pe.py şablonunun orijinal Çince metni + İngilizce çalışma
  çevirisi (birebir kullanılabilir).
- `references/model-notes.md` — model kartı + discussions/8'in distillenmiş notları
  (token matematiği, determinizm, ComfyUI token kontrol node'ları, offline PE yöntemleri).

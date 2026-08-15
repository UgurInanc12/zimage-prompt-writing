# Z-Image-Turbo Model Notları (model kartı + discussions/8 distillenmiş)

Kaynaklar: `https://huggingface.co/Tongyi-MAI/Z-Image-Turbo` (README) ve
`https://huggingface.co/Tongyi-MAI/Z-Image-Turbo/discussions/8` (PROMPTING GUIDE, resmi
takım yanıtları). İncelenme tarihi: 2026-08.

## Model ailesi (README'den)

- 6B parametre, **S3-DiT** (Scalable Single-Stream DiT): metin + görsel semantik tokenlar +
  VAE tokenları tek akışta birleşir → metin kompozisyona doğrudan işler, parametre verimliliği
  yüksek.
- Varyantlar: **Z-Image-Turbo** (distilled, 8 NFE, CFG'siz, sub-second/H800, 16GB VRAM'a
  sığar), Z-Image (temel, 50 adım, CFG + negatif prompt destekler, fine-tune için),
  Z-Image-Omni-Base, Z-Image-Edit (editing).
- Model Zoo tablosu: Turbo = Pre-Training ✅ SFT ✅ RL ✅, Step 8, **CFG ❌**, Diversity Low,
  Fine-Tunability N/A. (Temel Z-Image: Step 50, CFG ✅, Diversity Medium/High, fine-tune Easy.)
- Diffusers kullanımı: `guidance_scale=0.0`, `num_inference_steps=9` (= 8 DiT forward),
  bf16, `max_sequence_length=1024` lokal için.
- Metin render'ı: İngilizce + Çince **bilingual, doğru** render — modelin imza özelliği.
- Prompt Enhancing & Reasoning: resmi demo'da PE şablonu LLM'i çalıştırır → model
  "yüzey betimlemenin ötesine geçer, dünya bilgisine dokunur".

## Resmi takım yanıtları (discussions/8)

1. **Prompting**: "Z-Image-Turbo works best with long and detailed prompts. You may consider
   first manually writing the prompt and then feeding it to an LLM to enhance it." PE şablonu
   resmi olarak öneriliyor (pe.py).
2. **Negatif prompt**: "this is a few-step distilled model that does not rely on
   classifier-free guidance during inference... does not use negative prompts at all."
3. **LoRA**: aktif olarak üzerinde çalışılıyordu (discussion tarihinde).
4. **Token limiti**: demo 512 token (hız için); lokal 1024. "600-1000 words may result in
   800-1333 tokens roughly (0.75 word per token)". TE: Qwen3-4B.
5. **ComfyUI token kontrolü**: `ConditioningTruncate` (github.com/ClownsharkBatwing/RES4LYF),
   `CLIPTokenCounter` (github.com/pamparamm/ComfyUI-ppm) — kesme/uzunluk ölçümü için.
6. **Reasoning model dışında yapılır**: "the CLIP is only a translator" — PE/akıl yürütme
   LLM tarafında; ComfyUI'da ComfyUI-Ollama (stavsap/comfyui-ollama) veya ComfyUI-LLM-party
   ile offline yapılabilir.
7. **Dil**: topluluk bulgusu — aynı sahnenin İngilizce PE çıktısı Çince'den belirgin iyi;
   Z-Image İngilizce prompt'ları tercih ediyor (Çince render yine de mükemmel).
8. **Determinizm**: 600-1000 kelimelik aynı prompt neredeyse aynı render verir; cfg artırma
   işe yaramaz (CFG=0). Çeşitlilik = prompt'a anlamlı yeni detay eklemek + seed.
9. **İlham**: zimage.net/inspiration — tek tıkla kopyalanabilir prompt galerisi.

## Bu makinedeki ComfyUI kurulumu (image_z_image_turbo_int8.json)

- UNET: `z_image_turbo_int8_convrot.safetensors` (int8, 3090 için)
- TE: `qwen_3_4b_fp8_mixed.safetensors`, CLIPLoader type=`lumina2`
- VAE: `ae.safetensors`
- KSampler: steps=8, cfg=1, sampler res_multistep, scheduler simple, denoise=1
  (cfg=1 = ComfyUI'da CFG yok demektir — diffusers'taki guidance_scale=0 karşılığı)
- EmptySD3LatentImage: 1024x1024
- ConditioningZeroOut ile negative zero'lanıyor (Turbo'da negative kullanılmaz; şablon
  bunu zaten sıfırlıyor)
- Unload node'ları: KSampler öncesi (TE boşaltma) + VAEDecode öncesi (UNET boşaltma)

## LTX-2.5 ile temel farklar (karıştırmamak için)

| | LTX-2.5 | Z-Image-Turbo |
|---|---|---|
| Negatif prompt | Kullanılmaz (CFG=1) | **YOK — model görmez** |
| Metin render | Zayıf — post'a bırak | **Güçlü — tırnak içinde yaz** |
| Prompt uzunluğu | 4-8 cümle sahne | Uzun + detaylı (100-750 kelime) |
| Dil | İngilizce | İngilizce tercih (Çince render OK) |
| Meta etiketler | Kaçınılmalı | **YASAK** (PE kuralı) |
| Diyalog/tırnak | Konuşma tırnak içinde (video) | Görsel METİN tırnak içinde |
| Determinizm | Orta | Yüksek (çeşitlilik = yeni detay) |

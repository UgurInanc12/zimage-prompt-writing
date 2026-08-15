# Z-Image-Turbo Model Notes (distilled from model card + discussions/8)

Sources: `https://huggingface.co/Tongyi-MAI/Z-Image-Turbo` (README) and
`https://huggingface.co/Tongyi-MAI/Z-Image-Turbo/discussions/8` (PROMPTING GUIDE, official
team answers). Reviewed: 2026-08.

## Model family (from the README)

- 6B parameters, **S3-DiT** (Scalable Single-Stream DiT): text + visual semantic tokens +
  VAE tokens merge in a single stream → text feeds directly into composition, high
  parameter efficiency.
- Variants: **Z-Image-Turbo** (distilled, 8 NFE, CFG-free, sub-second/H800, fits in 16GB
  VRAM), Z-Image (base, 50 steps, supports CFG + negative prompt, for fine-tuning),
  Z-Image-Omni-Base, Z-Image-Edit (editing).
- Model Zoo table: Turbo = Pre-Training ✅ SFT ✅ RL ✅, Step 8, **CFG ❌**, Diversity Low,
  Fine-Tunability N/A. (Base Z-Image: Step 50, CFG ✅, Diversity Medium/High, fine-tune Easy.)
- Diffusers usage: `guidance_scale=0.0`, `num_inference_steps=9` (= 8 DiT forwards),
  bf16, `max_sequence_length=1024` for local.
- Text rendering: English + Chinese **bilingual, accurate** rendering (the model's
  signature feature).
- Prompt Enhancing & Reasoning: in the official demo the PE template drives an LLM → the
  model "goes beyond surface description, tapping world knowledge".

## Official team answers (discussions/8)

1. **Prompting**: "Z-Image-Turbo works best with long and detailed prompts. You may consider
   first manually writing the prompt and then feeding it to an LLM to enhance it." The PE
   template is officially recommended (pe.py).
2. **Negative prompt**: "this is a few-step distilled model that does not rely on
   classifier-free guidance during inference... does not use negative prompts at all."
3. **LoRA**: was actively being worked on (at the discussion date).
4. **Token limit**: demo 512 tokens (for speed); local 1024. "600-1000 words may result in
   800-1333 tokens roughly (0.75 word per token)". TE: Qwen3-4B.
5. **ComfyUI token control**: `ConditioningTruncate` (github.com/ClownsharkBatwing/RES4LYF),
   `CLIPTokenCounter` (github.com/pamparamm/ComfyUI-ppm) (for truncation/length
   measurement).
6. **Reasoning happens outside the model**: "the CLIP is only a translator" (PE/reasoning
   is on the LLM side; in ComfyUI it can be done offline with ComfyUI-Ollama
   (stavsap/comfyui-ollama) or ComfyUI-LLM-party).
7. **Language**: community finding (the English PE output of the same scene is clearly
   better than the Chinese one; Z-Image prefers English prompts, Chinese rendering is
   still excellent).
8. **Determinism**: the same 600-1000 word prompt gives nearly the same render; raising
   cfg does nothing (CFG=0). Variety = adding meaningful new details to the prompt + seed.
9. **Inspiration**: zimage.net/inspiration (a gallery of one-click copyable prompts).

## ComfyUI setup on this machine (image_z_image_turbo_int8.json)

- UNET: `z_image_turbo_int8_convrot.safetensors` (int8, for the 3090)
- TE: `qwen_3_4b_fp8_mixed.safetensors`, CLIPLoader type=`lumina2`
- VAE: `ae.safetensors`
- KSampler: steps=8, cfg=1, sampler res_multistep, scheduler simple, denoise=1
  (cfg=1 means no CFG in ComfyUI; the diffusers equivalent of guidance_scale=0)
- EmptySD3LatentImage: 1024x1024
- negative is zeroed with ConditioningZeroOut (negative is not used on Turbo; the
  template already zeroes it)
- Unload nodes: before KSampler (TE offload) + before VAEDecode (UNET offload)

## Key differences from LTX-2.5 (to avoid confusion)

| | LTX-2.5 | Z-Image-Turbo |
|---|---|---|
| Negative prompt | Not used (CFG=1) | **NONE (model never sees it)** |
| Text rendering | Weak (leave to post) | **Strong (write it in quotes)** |
| Prompt length | 4-8 sentence scene | Long + detailed (100-750 words) |
| Language | English | English preferred (Chinese rendering OK) |
| Meta tags | Should be avoided | **FORBIDDEN** (PE rule) |
| Dialogue/quotes | Speech in quotes (video) | Visual TEXT in quotes |
| Determinism | Medium | High (variety = new details) |

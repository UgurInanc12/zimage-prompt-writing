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

# Z-Image-Turbo Prompt Writing (official PE methodology)

## When to Use

- When writing prompts to generate images with Z-Image-Turbo (on this machine: the
  ComfyUI `image_z_image_turbo_int8` workflow).
- When the user gives a visual idea → produce the final prompt via the PE procedure.

Sources (reviewed 2026-08):
- Model card: `https://huggingface.co/Tongyi-MAI/Z-Image-Turbo`
- PROMPTING GUIDE (official team answers): `.../discussions/8`
- Official Prompt Enhancer (PE) template: Space `.../blob/main/pe.py`

## Model Facts (differences that shape the prompt: do NOT confuse with LTX)

- **No negative prompt**: few-step distilled (Decoupled-DMD + DMDR/RL), CFG-free
  inference. In Diffusers `guidance_scale=0.0`, in ComfyUI `cfg=1`. Writing a negative
  prompt is a waste of time, the model never sees it. (The Z-Image BASE model supports
  CFG + negative prompts; Turbo does not.)
- **Long and detailed prompts work best** (official advice: "long and detailed prompts").
- **Language: prefer English** (community-verified: the same scene comes out much better
  with an English PE output). Chinese rendering is strong (text to be rendered can be
  written in the prompt in either language).
- **Text rendering is a STRONG side**: the opposite of LTX (write any text that should
  appear in the image verbatim, in quotes, in the prompt; see PE step 4).
- **Token budget**: demo 512 tokens; local 1024 (`max_sequence_length=1024`).
  ~0.75 words/token → 512 tok ≈ 380 words; 1024 tok ≈ 750 words. TE = Qwen3-4B
  (`lumina2` type in ComfyUI).
- **High determinism**: the same prompt gives a similar render; for variety, add
  MEANINGFUL new details to the prompt (the cfg setting does nothing, it is already 0).
  Seed changes give limited variation.
- **Steps**: 9 API steps = 8 DiT forwards (`steps=8` in ComfyUI).
- **Architecture**: 6B S3-DiT (single-stream): text + visual tokens in a single stream;
  text feeds directly into composition.
- Inspiration gallery: `https://zimage.net/inspiration` (copyable prompts).

## Official PE (Prompt Enhancer) Methodology: 4 steps

My job is to personally play the role from pe.py: the "visionary artist in a logic
cage" (produce an ultimate visual description that is faithful to the user's intent,
full of detail, aesthetic, and directly usable by T2I).

1. **Lock the core elements** (the IMMUTABLE elements of the user's prompt): subject,
   count, action, state + any specified IP names, colors, texts. These are absolutely
   preserved.
2. **Generative reasoning decision** (when the request is not a direct scene but
   requires a solution, e.g. "what is X?", "design", "show the solution to this
   problem"): first build a COMPLETE, concrete, visualizable plan in your mind; the
   description is built on top of it.
3. **Professional aesthetic + realism injection** (five axes):
   composition · light/shadow atmosphere · material/texture · color scheme · layered
   spatial depth.
4. **Precise handling of text elements (CRITICAL)**:
   - Write ALL text that should appear in the final image verbatim, in English double
     quotes: `"..."` (an explicit generation instruction).
   - For design work like posters/menus/UI: all text + font + typography/layout.
   - Text on signs/screens/road signs in the scene: content + position, size, material.
   - Text you added in your own reasoning (charts, solution steps): same rules.
   - If there is no text → all energy goes to pure visual detail.

## Output Rules (PE's strict requirements)

- Objective and concrete. Metaphors FORBIDDEN, emotional rhetoric FORBIDDEN.
- Meta tags like `"8K"`, `"masterpiece"` and drawing instructions FORBIDDEN.
- Output ONLY the final prompt (no explanation, no preamble, no alternatives).

## Prompt Construction Procedure (idea → final prompt)

1. Take the idea → lock the core elements (subject/count/action/color/text).
2. Is "generative reasoning" needed? (design/solution requests) → first a visualizable
   plan.
3. Add aesthetic detail on the 5 axes (composition, light, texture, color, depth).
4. Text audit: if the image has text, write it verbatim in quotes (+font/layout if
   needed).
5. Write in English; stay within the token budget (target ~100-380 words; long jobs up
   to 750).
6. Do NOT write a negative prompt. Do not use meta tags.
7. ComfyUI parameters (this machine): steps=8, cfg=1, 1024x1024, sampler res_multistep /
   scheduler simple, TE `qwen_3_4b_fp8_mixed` (lumina2), UNET `z_image_turbo_int8_convrot`.

## Example style (from the README: official showcase prompt)

> Young Chinese woman in red Hanfu, intricate embroidery. Impeccable makeup, red floral
> forehead pattern. Elaborate high bun, golden phoenix headdress, red flowers, beads. Holds
> round folding fan with lady, trees, bird. Neon lightning-bolt lamp (⚡️), bright yellow glow,
> above extended left palm. Soft-lit outdoor night background, silhouetted tiered pagoda
> (西安大雁塔), blurred colorful distant lights.

Note: short sentences, concrete objects, light + background detail, Chinese text directly
in the prompt.

## Supporting files

- `references/pe-template.md` (original Chinese text of the pe.py template + English
  working translation, usable verbatim).
- `references/model-notes.md` (distilled notes from the model card + discussions/8: token
  math, determinism, ComfyUI token control nodes, offline PE methods).

---
layout: archive
title: "How Frontier LLMs See: Soft Tokens, Pixel Shuffle, Transfusion and the Multimodal Recipe"
date: 2026-11-13
permalink: /posts/2026/11/how-frontier-llms-see
excerpt: "A field guide to multimodal LLMs: how image, video and audio vectors enter a text model with a fixed vocabulary, why token compression and the projector are different jobs, how Transfusion generates images, and the recipe patterns every frontier lab converged on."
tags:
- multimodal
- vision-language-models
- large-language-models
- transformers
- diffusion
- reinforcement-learning
---

Every frontier model now reads screenshots, PDFs, video and speech. Yet the underlying machine is still a language model with a fixed vocabulary of 150–250K token IDs. How does a 1080p screenshot fit into that? Why does a model that "sees" a document still lose points on AIME? And what is actually different between a model that understands images and one that draws them?

This post works through the mechanics from the first layer up, then zooms out to the training recipe and the patterns the major labs converged on in 2025–2026. I use NVIDIA's [Nemotron 3 Nano Omni](https://arxiv.org/abs/2604.24954) as the running example because its report is unusually detailed, but the plumbing is the same almost everywhere.

---

### 1. Everything Becomes Tokens

The LLM never sees pixels or waveforms. It sees **vectors in positions where word embeddings would have been**, and training makes those vectors mean something. Each modality gets its own path into one shared sequence:

```text
INPUT              ENCODER                       PROJECTOR          SEQUENCE
"Is revenue up?" → tokenizer (BPE → IDs)       → embedding table  → ~10 tokens
1920×1080 image  → ViT, 16 px patches          → shuffle + MLP    → ~2,040 tokens
60 s video, 2fps → same ViT + Conv3D tubelets  → shuffle + MLP    → ~15,360 tokens
60 s speech      → audio encoder (80 ms/token) → MLP              → 750 tokens
                                                                     │
                                                    one interleaved sequence
                                                                     ▼
                                                         LLM decoder → text, code, tool calls
```

Two numbers worth remembering: a 1080p screenshot costs about **2,000 tokens**, while the question about it costs maybe 10. One minute of video at 2 fps costs about **15,000**. Token efficiency is not a detail in multimodal models. It is the central engineering problem.

---

### 2. How Do Vectors Get Past a Fixed Vocabulary?

This is the question that confused me first. If the tokenizer defines a vocabulary of $V$ IDs, how can 2,040 image vectors enter the sequence?

The answer: **the vocabulary limits what can be _looked up_, not what can _enter_.** Layer 1 of a transformer reads one tensor of shape $\text{seq} \times d$. A text token fills its row by looking up the $V \times d$ embedding table. An image fills its rows with vectors from the projector. The tokenizer only needs a few reserved IDs to mark *where* those rows go.

```text
input_ids      <img_start> <img> <img> <img> … <img> <img_end>  A    red  bus  .
                           └──── N copies of one placeholder ID ───┘
E[input_ids]   E[·]       E[img] E[img] E[img] … E[img] E[·]     E[·] E[·] E[·] E[·]
inputs_embeds  E[·]        s₁    s₂    s₃   …  s_N   E[·]     E[·] E[·] E[·] E[·]
                           ▲─────── projector(ViT(pixels)) ──────▲
labels         -100       -100  -100  -100  … -100  -100      A    red  bus  .
```

In code (Hugging Face style):

```python
emb    = model.get_input_embeddings()(ids)          # (seq, d): every slot is a table lookup
mask   = (ids == IMG_ID)[..., None].expand_as(emb)
emb    = emb.masked_scatter(mask, projector(vit(pixels)))  # overwrite the N slots, in order
labels = ids.masked_fill(ids == IMG_ID, -100)       # cross_entropy's ignore_index skips them
out    = model(inputs_embeds=emb, labels=labels)    # layer 1 cannot tell the two kinds apart
```

A few details that matter:

1. **The preprocessor computes $N$ before tokenizing.** It knows the image size and the patch rules, so the text sequence already has exactly the right number of placeholder slots.
2. **`masked_scatter`** fills positions where the mask is true with consecutive values from the source tensor. The placeholder row of the embedding table is overwritten, so it never affects the output and receives no gradient.
3. **`-100` is PyTorch's default `ignore_index`** for `cross_entropy`. Image positions are dropped from the loss because a soft token has no correct "next ID" to score.
4. **Image slots still receive gradient as context.** Later text tokens attend to them, so the loss on "red bus" flows back through attention into the projector and encoder. That is how they learn.

Names vary by lab: Qwen uses `<|vision_start|>`, `<|image_pad|>`, `<|vision_end|>`; Gemma uses `<start_of_image>`, `<image_soft_token>`, `<end_of_image>`. And the embedding table itself? It is an ordinary parameter matrix trained by backprop with the rest of the model; only rows used in a batch get gradients. The BPE merges of the tokenizer, by contrast, are fixed before training and never touched by gradients.

#### How many special tokens do tokenizers carry?

Counting the published tokenizer files of recent open models:

| Model | Named special tokens | Reserved or spare IDs |
|---|---|---|
| Qwen3.5 | 33 | 243 spare embedding rows beyond the tokenizer |
| Kimi | 16–23 | a 256-ID reserved block |
| DeepSeek V4.1 | ≈62 | 799 placeholders + 422 multimodal-span placeholders |
| Gemma 4 | 24 | 6,233 `<unusedN>` |
| Llama 4 | 20 | 1,115 reserved |
| Mistral (Tekken) | 26 | IDs 0–999 for control tokens |
| Nemotron 3.5 Super VL | 30 | IDs 0–999, Tekken-style |
| gpt-oss | 10 | 11 reserved |

Reserved rows let a lab add new markers (an image span, a tool call, a timestamp) later without resizing the embedding table or the LM head. A handful of named tokens carries all the multimodal plumbing: the image itself costs thousands of *soft tokens*, not thousands of vocabulary entries.

---

### 3. Soft Tokens

A **hard token** is an ID that selects one row of the embedding table. A **soft token** is any $d$-dimensional vector placed at an input position *without* a table lookup.

```text
hard token:  ID 3923 → E[3923]                 one of V fixed points
mixture:     p · E   (p = distribution over V)  a weighted blend of rows
soft token:  any v ∈ ℝ^d                        computed by a network, or learned directly
```

The idea predates VLMs. In **prompt tuning** ([Lester et al., 2021](https://arxiv.org/abs/2104.08691)) you freeze the whole LLM, prepend $k$ trainable vectors to every input, and train only those. **Prefix tuning** ([Li & Liang, 2021](https://arxiv.org/abs/2101.00190)) does the same for keys and values at every layer. The learned vectors work, but if you decode each one to its nearest word you mostly get noise. The model does not need its inputs to be words; it needs vectors whose effect on its layers is useful.

Image tokens are the same thing, conditioned on the input. Look at the **projector warmup** stage of any VLM recipe: the ViT and LLM are frozen, and gradient descent shapes the vectors fed into the LLM so its text predictions improve. That *is* prompt tuning, except a network computes the soft prompt from each image instead of storing it as a parameter. BLIP-2 describes its Q-Former outputs as "soft visual prompts", and Gemma literally named its placeholder `<image_soft_token>`.

| | Token embedding (hard) | Soft token |
|---|---|---|
| Comes from | `E[id]`, one row of the $V \times d$ table | any computation: projector, learned parameter, hidden state |
| Possible values | one of $V$ fixed vectors | anywhere in $\mathbb{R}^d$, often far from every word |
| Has an ID or string | yes | no; a placeholder ID only marks the slot |
| Model can output it | yes, the LM head scores it | no, input only |
| Information per position | one of ~150–250K choices, ≈17–18 bits | $d$ continuous numbers: enough to summarise a 32×32 px region |

So is a soft token "an embedding"? In the broad sense, yes: it is a vector at an input position. In the narrow sense (a row of the embedding table), no. A useful edge case: if you add 64 new vocabulary entries and train only their rows, you have nearly reproduced prompt tuning. The difference is that those rows have IDs and, with tied embeddings, the LM head can now *predict* them. **A token embedding is a vector the model can both read and write; a soft token is one it can only read.**

The same idea shows up elsewhere: gist and memory tokens (a long context squeezed into a few vectors), Ovis's probability-weighted "visual vocabulary", and Coconut's continuous chain of thought, which feeds the last hidden state back in as the next input instead of a sampled word.

---

### 4. Four Ways to Wire Vision into an LLM

Every design answers one question: *where do visual features meet the language model?*

| Pattern | Examples | For | Against |
|---|---|---|---|
| **a. Projector (late fusion)** | LLaVA, Qwen-VL, InternVL, Nemotron | reuses a strong text LLM; simplest; the open default | every visual token sits in the sequence, so cost grows with resolution and frames |
| **b. Cross-attention** | Flamingo, Llama 3.2 Vision | LLM sequence stays short; LLM can stay frozen | extra layers; awkward with many interleaved images |
| **c. Discrete early fusion** | Chameleon, Emu3 | one vocabulary, one loss; can output images | quantized codes lose fine detail: bad for OCR, charts, documents |
| **d. Unified understand + generate** | Transfusion, Janus, BAGEL | one backbone reads and draws | two objectives compete for capacity; harder to train and serve |

One distinction is worth stating loudly: **"native multimodal" usually describes the training _schedule_, not the architecture.** Llama 4 and InternVL3 call themselves natively multimodal and still use a ViT encoder plus projector. "Native" means vision enters at pretraining rather than being bolted on afterwards.

---

### 5. The Vision Path: Token Compression vs. the Projector

The standard pipeline is often summarised as:

```text
ViT encoder → token compression → MLP projector → LLM
```

People (including me, at first) blur the middle two boxes. They act on **different axes of the same tensor**:

```text
ViT output            token compression             MLP projector
(N, d_vit)       →    (N/4, 4·d_vit)           →    (N/4, d_llm)
4,096 × 1,280         1,024 × 5,120                  1,024 × 4,096
                      count changes                  width changes; vectors move into
                      (mixes neighbouring tokens)    the LLM's embedding space
                                                     (each token handled separately)
```

| | Token compression | MLP projector |
|---|---|---|
| Changes | sequence length $N$ | width $d$ and coordinate system |
| Why it exists | cost: attention, KV cache and context all grow with $N$ | compatibility: the LLM only reads vectors shaped like its own embeddings |
| Operates | across tokens | per token, so $N$ in means $N$ out |
| Parameters | often none | yes; the first thing trained when attaching an encoder |
| Required? | no (LLaVA-1.5 sends all 576 patches through) | yes |

#### Pixel shuffle is one kind of compression

**Pixel shuffle** (space-to-depth) is the compression used by InternVL, Kimi and Nemotron. It is a reshape with no weights: every 2×2 block of neighbouring patch vectors is concatenated into one vector four times wider.

```python
x = vit(pixels)                                          # (B, H, W, d_vit), e.g. 64×64×1280
x = x.view(B, H//2, 2, W//2, 2, d_vit).permute(0, 1, 3, 2, 4, 5)
x = x.reshape(B, (H//2) * (W//2), 4 * d_vit)             # 4,096 → 1,024 tokens, no weights
mlp1 = nn.Sequential(nn.LayerNorm(4 * d_vit), nn.Linear(4 * d_vit, d_llm),
                     nn.GELU(), nn.Linear(d_llm, d_llm))  # projector, as in InternVL
tokens = mlp1(x)                                         # (B, 1,024, d_llm): soft tokens
```

The token count drops 4×, but every number survives the reshape. The real squeezing happens in the trained MLP, which decides what to keep when mapping $4 d_{\text{vit}} \to d_{\text{llm}}$. Other options fill the same slot and lose different things:

| Compression method | How it cuts tokens | What it loses | Used by |
|---|---|---|---|
| Pixel shuffle | concatenates 2×2 neighbours | nothing in the reshape; the MLP decides | Nemotron, InternVL, Kimi; Qwen's PatchMerger |
| Pooling | averages neighbours | fine detail | Gemma 3 (4,096 patches → 256 tokens) |
| Learned queries (Q-Former, Perceiver) | fixed set of queries cross-attends to all patches | anything beyond a fixed budget | BLIP-2, Flamingo, MiniCPM-V |
| Strided convolution | learned downsampling | some, but learned | Honeybee C-Abstractor, DeepSeek-OCR (16×) |
| Temporal grouping | Conv3D tubelets: 2 frames → one set of tokens | fast motion within a pair | Nemotron Omni video |
| Pruning / merging | drops redundant tokens; count depends on content | whatever the redundancy test misjudges | Efficient Video Sampling, ToMe |

The field moved off learned-query resamplers because 32–64 fixed queries cannot hold the small text on a document page. The useful cost metric is **pixels per LLM token** = patch size × merge factor. Nemotron Omni is 16 px × 2 = 32 px per token side; labs range from 16 px (Pixtral) to 48 px.

Why does the projector exist even when the widths happen to match? Because the encoder was trained with a contrastive or distillation objective and lives in its own space, with its own scale and geometry. LLaVA-1 used a single linear layer; LLaVA-1.5's two-layer MLP did noticeably better, which suggests the translation is not purely linear.

#### The rest of the vision path

- **Encoders.** CLIP is good at global semantics and weak on small text and positions. SigLIP 2 is the open default. DINOv2 has dense spatial features but no language alignment. NVIDIA's C-RADIO distills several teachers (CLIP-family, DINO, SAM) into one backbone. Unfreezing the encoder helps OCR and grounding but needs a lower learning rate than the LLM.
- **Resolution.** *Tiling* cuts images into fixed tiles plus a thumbnail, but seams cut through words. *Native dynamic resolution* (NaViT, Qwen2-VL onward, Nemotron Omni with 1,024–13,312 patches) encodes at the image's own aspect ratio with a variable token count.
- **Position.** A flat 1D RoPE puts the token above another hundreds of positions away. Fixes: M-RoPE (separate rotary dimensions for time, height, width), time-aligned variants like TMRoPE for audio and video, or simply writing timestamps as text (`<3.0 seconds>`), which Qwen switched to for long video.

---

### 6. Video, Audio and the Token Budget

Video is why token efficiency became a first-class research problem: a frame costs about as much as an image, and there are thousands of frames. Nemotron Omni stacks three tricks:

1. **Sample frames** and randomly vary patches per frame in training so the model learns to trade resolution for frames.
2. **Fuse time in the encoder.** A Conv3D patch embedder merges every 2 consecutive frames into one tubelet: half the tokens.
3. **Prune at inference.** Efficient Video Sampling drops tokens that barely changed from the previous tubelet at the same position.

For 512 frames at 512×512 this goes from ≈141K tokens to ≈75K (Conv3D) to ≈42K (plus EVS at $q=0.5$), for about half a point of accuracy. Audio is cheap by comparison: a FastConformer encoder with 8× subsampling gives one token per 80 ms, 12.5 tokens per second, so more than five hours of speech fits in a 256K context.

---

### 7. Generating Images: Transfusion

Everything above is about *reading*. Meta's [Transfusion](https://arxiv.org/abs/2408.11039) (Zhou et al., 2024) is the reference design for a model that both reads and draws.

Chameleon turned images into discrete VQ codes so a single cross-entropy loss covered everything, and lost detail to quantization. Transfusion keeps images continuous: text gets next-token prediction, image patches get a **diffusion loss**, and one transformer with shared weights trains on both in the same sequence.

```text
sequence:  The  cat  <BOI>  [ p₁  p₂  …  p₂₅₆ ]  <EOI>  is  orange
           ───────────────  ────────────────────  ───────────────
loss:      next-token CE    diffusion MSE          next-token CE
                            (predict the added noise)
input:     E[id] lookups    noisy VAE latent patches → linear or U-Net down block → d
```

$$\mathcal{L} = \mathcal{L}_{\text{LM}}(\text{text}) + \lambda \cdot \mathcal{L}_{\text{diffusion}}(\text{image}), \qquad \lambda = 5$$

The attention mask is causal across the sequence but **bidirectional inside each image**:

```text
             The  cat  BOI  p₁  p₂  p₃  EOI  is
The           ■
cat           ■    ■
BOI           ■    ■    ■
p₁            ■    ■    ■    ■   ■   ■        ← patches see each other
p₂            ■    ■    ■    ■   ■   ■
p₃            ■    ■    ■    ■   ■   ■
EOI           ■    ■    ■    ■   ■   ■   ■
is            ■    ■    ■    ■   ■   ■   ■   ■
```

Patches of one image have no natural order and are denoised together; text must stay causal so it can still be generated. In the paper's ablation, making attention causal inside images hurt generation.

```python
x0   = vae.encode(img)                                   # continuous latents
t    = randint(0, 1000); eps = randn_like(x0)
xt   = sqrt(abar[t]) * x0 + sqrt(1 - abar[t]) * eps      # noised to step t
img  = patch_in(patchify(xt)) + t_emb(t)                 # soft tokens
h    = transformer(cat(E[text_ids], img), mask=causal_plus_image_blocks)
loss = ce(lm_head(h[text_pos]), next_ids) + 5 * mse(patch_out(h[img_pos]), eps)
```

**Inference switches between two modes.** In text mode the model samples tokens as usual. When it emits `<BOI>`, it appends $N$ patches of pure noise and runs a denoising loop (250 steps in the paper, trained on 1,000 timesteps) with the text prefix cached. Then it appends `<EOI>` and returns to text, with the finished image in context, so it can caption or discuss what it drew.

The results were strong: a 7B model trained from scratch on 2T tokens generated images and text on par with similar-scale diffusion models and language models. It beat Chameleon at text-to-image with less than a third of the compute and matched Chameleon's FID with 34× less. With U-Net down/up blocks an image can shrink to just 16 patches.

#### The catch: one representation, two jobs

VAE latents are built to reconstruct pixels, which suits generation. Understanding wants semantic features like SigLIP's. One representation serves both imperfectly, so most unified models since then split the paths:

| Model | How it splits understanding from generation |
|---|---|
| Janus / Janus-Pro (DeepSeek) | SigLIP for understanding, a VQ tokenizer for generation |
| BAGEL (ByteDance Seed) | ViT + VAE inside a mixture-of-transformers with separate expert weights per mode |
| LMFusion (Meta) | frozen Llama text weights plus parallel image-specific weights |
| MetaQuery (Meta), BLIP3-o (Salesforce) | keep an understanding VLM; it conditions a separate diffusion model |

There is also a cheaper route for many "generation" tasks: write code that a tool renders, look at the screenshot, critique and fix. Documents, PDFs, charts, slides, 3D scenes and games all have a program behind them. Pixel-level generation earns its cost when the output has no program: photos, natural images, edits of real images.

---

### 8. The Training Stack

Almost every frontier multimodal model passes through the same stages; they differ in what goes where.

| Stage | What the model learns | Typical failure |
|---|---|---|
| Text pretraining | language, knowledge, code | — |
| Alignment warmup | map encoder space into LLM space; only the projector trains | skip it and a random projector sends garbage gradients into the LLM |
| Multimodal pretraining / mid-training | perception at scale: OCR, charts, grounding, documents, video | text regression if the mix is wrong |
| SFT | instruction following, reasoning format, tool use | short-answer bias; capped by teacher quality |
| Preference and RL | correctness, abstention, grounding, agentic skill | reward hacking, refusal collapse, forgetting |
| Distill and quantize | serve it cheaply | per-benchmark drops hidden by averages |

**The forgetting tax is real.** Millions of short-answer perception samples pull a model toward terse answers, and RL on images moves shared weights. Nemotron Omni against its own text backbone: AIME25 89.1 → 82.1, LiveCodeBench 68.3 → 63.2, while instruction following and long-context scores actually went up. The Omni recipe uses seven SFT stages (each new encoder gets a projector-only warmup before it is unfrozen; context grows 16K → 48K → 256K) and then five RL stages, ending with a text-only RL stage specifically to restore text skills after multimodal RL.

---

### 9. What the Frontier Converged On

Reading the 2025–2026 reports side by side (Qwen, Kimi, DeepSeek, GLM, MiniMax, Meta's Muse, Google's Gemini and Gemma, Xiaomi's MiMo, NVIDIA's Nemotron), seven patterns repeat:

1. **Vision from the first pretraining tokens.** Kimi found a constant 10% vision share from the start beat a 50:50 mix added late. Apple's study of 457 models found no inherent advantage to late fusion.
2. **In-house encoders, trained less like CLIP.** Kimi K3 and MiMo train ViTs from scratch with next-token loss; NVIDIA (C-RADIO) and Zhipu distill several teacher encoders into one.
3. **Perception as test-time search.** The model crops, zooms and rotates with code inside its reasoning, trained with RL. With a crop or Python tool, the top closed models all land at 83–86 on ScreenSpot-Pro: tools close the GUI-grounding gap.
4. **Specialists, then on-policy distillation.** Train domain teachers separately, then distill them into one student on the student's own samples. Xiaomi's **MOPD** (multi-teacher on-policy distillation, MiMo-V2-Flash) uses a per-token advantage $\text{sg}[\log \pi_T - \log \pi_S]$ and recovered more of the teachers' headroom than mixed multi-domain RL or weight merging. Kimi K3, Keye-VL 2.0 and Nemotron 3.5 Super VL use it.
5. **Linear-attention backbones for long visual context.** Mamba (Nemotron), Gated DeltaNet at 3:1 (Qwen3.5), KDA (Kimi K3), sparse plus linear attention (GLM, MiniMax).
6. **Time written as text.** Plain timestamps replaced absolute-time position IDs, which grew large and sparse on long video.
7. **The token budget is a dial.** Gemini's media resolution, Gemma's 70–1,120 budget, OpenAI's detail levels, NVIDIA's EVS: documents need the high setting; most reasoning doesn't.

---

### 10. Questions I Now Use to Check Understanding

If you want to test whether someone (or yourself) actually understands multimodal LLMs rather than the vocabulary around them, these are the questions I find most revealing:

1. **The tokenizer has ~150K IDs. How do 2,000 image vectors enter the sequence, and what loss do those positions get?** (Placeholders, `masked_scatter`, `-100`, gradient as context.)
2. **Why can't the MLP projector reduce the number of image tokens?** (It is applied per token: $N$ in, $N$ out. Reducing $N$ needs cross-token mixing.)
3. **What is a soft token, and how does projector warmup relate to prompt tuning?** (Same game; the soft prompt is computed per image.)
4. **Why does Transfusion use bidirectional attention inside an image, and why did later unified models split their encoders?** (No natural patch order; representation conflict.)
5. **Walk a 1920×1080 screenshot from pixels to the LLM's first layer. How many tokens?** (≈8K patches → ≈2K tokens; the same screen as text might be 300.)
6. **Your VLM is 7 points worse than its text backbone on AIME. Diagnose it.** (Short-answer bias from perception SFT, shared-weight drift from image RL; fixes include text replay, text-RL at the end, distillation from the text teacher.)
7. **Vendors report OSWorld scores from 47 to 85. How do you compare them?** (Different versions, scoring rules and harnesses; rerun in your own.)

---

### References

- Nemotron 3 Nano Omni technical report, [arXiv 2604.24954](https://arxiv.org/abs/2604.24954)
- [LLaVA](https://arxiv.org/abs/2304.08485) · [InternVL 1.5](https://arxiv.org/abs/2404.16821) · [Qwen2-VL](https://arxiv.org/abs/2409.12191) · [Flamingo](https://arxiv.org/abs/2204.14198) · [BLIP-2](https://arxiv.org/abs/2301.12597)
- [Chameleon](https://arxiv.org/abs/2405.09818) · [Transfusion](https://arxiv.org/abs/2408.11039) · [Scaling laws for native multimodal models](https://arxiv.org/abs/2504.07951)
- [Janus](https://arxiv.org/abs/2410.13848) · [Janus-Pro](https://arxiv.org/abs/2501.17811) · [BAGEL](https://arxiv.org/abs/2505.14683) · [LMFusion](https://arxiv.org/abs/2412.15188) · [MetaQuery](https://arxiv.org/abs/2504.06256) · [BLIP3-o](https://arxiv.org/abs/2505.09568)
- [Prompt tuning](https://arxiv.org/abs/2104.08691) · [Prefix tuning](https://arxiv.org/abs/2101.00190)
- [MiMo-V2-Flash (MOPD)](https://arxiv.org/abs/2601.02780) · [GSPO](https://arxiv.org/abs/2507.18071) · [Efficient Video Sampling](https://arxiv.org/abs/2510.14624) · [MMLongBench-Doc](https://arxiv.org/abs/2407.01523)

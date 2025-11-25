# Norwegian ASR Reading List – Model Architectures & Systems

Reading list focused on post-Whisper architectures, with Parakeet-style models in mind. Use this as a living document: each entry is designed so you can quickly decide if it should be “next up” in the reading group.

---

## 1. Whisper: Robust Speech Recognition via Large-Scale Weak Supervision  
**Radford et al., 2022**  
**Link:** https://arxiv.org/abs/2212.04356  

**Why it matters**

This is the reference point for almost everything else on this page. Whisper shows that a single encoder–decoder Transformer, trained on hundreds of thousands of hours of noisy web audio with weak supervision (subtitles, web captions, etc.), can give robust zero-shot ASR across many languages and domains. It also establishes the “30s chunk + joint ASR/ST + multitask prompts” recipe that later models either copy or deliberately move away from.  [oai_citation:0‡arXiv](https://arxiv.org/abs/2212.04356?utm_source=chatgpt.com)  

For your team, Whisper is the baseline *design space*: log-Mel input, aggressive time subsampling, large decoder, and a multitask token vocabulary that blends language, task, and timestamp control. Even if you don’t adopt the full multitask setup, you’ll constantly be asking “how does this compare to Whisper-large-v3 on FLEURS / MLS / CV?” in both accuracy and throughput.

**What to focus on**

- The data section: how they embrace noisy, weak labels at scale rather than curated corpora.  
- The inference strategy for long-form decoding (sequential chunking with timestamp prediction).  
- The multitask tokenization design (task tokens, language tokens, timestamps).  

Use this as the conceptual “origin” when you read newer work that optimizes speed (Parakeet, Moonshine) or openness (MMS, OWSM, Granary).

---

## 2. Fast Conformer with Linearly Scalable Attention for Efficient Speech Recognition  
**Shi et al., 2023 (Google)**  
**Link:** https://arxiv.org/abs/2305.05084  

**Why it matters**

Fast Conformer is the workhorse encoder behind many modern high-throughput ASR systems (including Parakeet-style models). It keeps the Conformer’s strong accuracy but replaces quadratic self-attention with a linearly scalable scheme that mixes local and limited global attention, plus efficient convolutional modules. The result: similar or better WER at a fraction of the compute, especially for long-form input.  [oai_citation:1‡arXiv](https://arxiv.org/abs/2305.05084?utm_source=chatgpt.com)  

For your project, this paper essentially answers “what encoder do we want if we care about both accuracy and throughput?” It’s particularly relevant if you aim for 600M–1B parameter models that can still process hours of audio quickly.

**What to focus on**

- The exact attention structure (how they achieve linear scaling, the local windowing, global tokens).  
- Downsampling strategies and how they affect latency vs. accuracy.  
- Comparisons against vanilla Conformer and other encoders on large-scale benchmarks.  

If you’re designing a Parakeet-like Norwegian ASR model, this is the canonical reference for your encoder block design and ablation baselines.

---

## 3. Efficient Sequence Transduction by Jointly Predicting Tokens and Durations (TDT)  
**Zhang et al., 2023 (NVIDIA + Suno)**  
**Link:** https://arxiv.org/abs/2304.06795  

**Why it matters**

This paper introduces Token-Duration Transducers (TDT), a variant of RNN-T that explicitly predicts both *which* token and *for how long* it should last. Instead of emitting a long sequence of repeated blank/non-blank steps, TDT compresses alignments into token–duration pairs. This cuts down the length of alignment lattices and significantly reduces decoding cost, while preserving the advantages of transducer-style streaming.  [oai_citation:2‡arXiv](https://arxiv.org/abs/2304.06795?utm_source=chatgpt.com)  

The architecture is central to Parakeet-TDT: a Fast Conformer encoder feeding a TDT (or similar) decoder. For your team, this is the key to ultra-fast, streaming-friendly ASR that still benefits from strong encoder representations.

**What to focus on**

- How the TDT loss is defined and how it reduces computational complexity compared to classic RNN-T.  
- How durations are modeled and constrained (e.g., max duration, discretization).  
- Empirical trade-offs between TDT and RNN-T/CTC in latency and WER.  

If you’re currently CTC-based, this paper is the “do we move to TDT/RNN-T?” decision point.

---

## 4. Canary‑1B‑v2 & Parakeet‑TDT‑0.6B‑v3: Efficient and High‑Performance Models for Multilingual ASR and AST  
**Sekoyan et al., 2025 (NVIDIA)**  
**Link:** https://arxiv.org/abs/2509.14128  

**Why it matters**

This is *the* Parakeet/Canary paper: it describes how NVIDIA builds multilingual FastConformer–TDT models that top open leaderboards while staying very fast. Canary‑1B‑v2 is a multi-task ASR+AST AED model; Parakeet‑TDT‑0.6B‑v3 is a 600M-parameter multilingual ASR model in the same ecosystem. Both are trained on ~1.7M hours of data from sources like Granary and NeMo ASR Set 3.0, with careful data balancing and explicit use of non-speech audio to reduce hallucinations.  [oai_citation:3‡arXiv](https://arxiv.org/abs/2509.14128?utm_source=chatgpt.com)  

You essentially get a blueprint for the system you want to build for Norwegian: Fast Conformer encoder, TDT decoder, multilingual tokenizer, and a hierarchy of training stages (pre-training + multilingual fine-tuning). The paper also compares Fast Conformer vs. an nGPT encoder on massive data, which is useful if you ever consider replacing the encoder.

**What to focus on**

- Architecture details of FastConformer+TDT in a production-scale multilingual setting.  
- The data mixture (Granary + NeMo ASR sets), balancing strategy, and use of non-speech audio.  
- Timestamp generation via an auxiliary CTC model and NeMo Forced Aligner.  
- Comparisons against Whisper-large-v3 and SeamlessM4T in both accuracy and speed.  

Read this after Fast Conformer + TDT; it shows how the pieces come together at scale.

---

## 5. Scaling Speech Technology to 1,000+ Languages (Massively Multilingual Speech, MMS)  
**Pratap et al., 2023 (Meta)**  
**Link:** https://arxiv.org/abs/2305.13516  

**Why it matters**

MMS demonstrates end-to-end self-supervised pre-training over an enormous multilingual corpus (readings of religious texts for >1,000 languages), producing wav2vec 2.0 encoders and downstream ASR/TTS models that cover 1,107 languages.  [oai_citation:4‡arXiv](https://arxiv.org/abs/2305.13516?utm_source=chatgpt.com)  

The architecture isn’t exotic, but the *scaling regime* and data strategy are crucial. For a Norwegian-focused model, MMS is the natural reference for: (1) how far you can get with SSL on unlabeled speech, and (2) how to handle extreme language imbalance with temperature-based sampling and language-specific fine-tuning.

**What to focus on**

- How they construct and clean religious-text speech corpora across so many languages.  
- The SSL pre-training recipe (wav2vec 2.0 variants, masking strategy, model sizes).  
- Fine-tuning strategies for very low-resource languages and LID.  

This is a good paper to read when you’re thinking about extending your Norwegian model to related Scandinavian or minority languages.

---

## 6. Automatic Speech Recognition in the Modern Era: Architectures, Training, and Evaluation  
**Nayem et al., 2025 (Survey)**  
**Link:** https://arxiv.org/abs/2510.12827  

**Why it matters**

This is a broad survey that contextualizes Whisper-style models, Parakeet/Canary, MMS, and speech-LLMs in a single narrative. It covers encoder choices (Conformer, Fast Conformer, E-Branchformer, etc.), decoder families (CTC, RNN‑T, AED, TDT), and training paradigms (supervised, semi/weakly supervised, SSL). It also discusses evaluation issues, including long-form robustness and multilingual benchmarks.  [oai_citation:5‡alphaXiv](https://www.alphaxiv.org/?subcategories=audio-and-speech-processing&utm_source=chatgpt.com)  

For your group, this is a useful “index”: once everyone has roughly aligned on the Whisper + FastConformer/TDT paradigm, this survey helps identify gaps—e.g. streaming vs. offline performance, low-resource languages, or integration with LLMs.

**What to focus on**

- Taxonomy of ASR model families and where Parakeet-style systems sit.  
- Sections on large-scale training and data regimes post-Whisper.  
- Discussion of robustness (noise, accents, long-form, domain shift) and evaluation pitfalls.  

Skim this early and revisit whenever you need a map of the current model landscape.

---

## 7. Open ASR Leaderboard: Towards Reproducible and Transparent Multilingual and Long‑Form Speech Recognition Evaluation  
**Gandhi et al., 2025 (Hugging Face + collaborators)**  
**Link:** https://arxiv.org/abs/2510.06961  

**Why it matters**

This paper defines a standard benchmark and tooling for evaluating ASR models across multilingual and long-form datasets, including WER and throughput metrics. It underpins the Hugging Face Open ASR Leaderboard, where Whisper, Parakeet, Canary, Granite Speech and others are compared under the same scripts and test sets.  [oai_citation:6‡arXiv](https://arxiv.org/abs/2510.06961?utm_source=chatgpt.com)  

For you, this is both an evaluation recipe and a reality check: it tells you which datasets matter, how to measure real-time factor (RTFx), and how your future Norwegian model might be positioned against current open models.

**What to focus on**

- Dataset selection, particularly multilingual and long-form corpora.  
- Evaluation protocol (normalization, punctuation handling, diarization assumptions).  
- How they report trade-offs between accuracy and speed, and how this affects leaderboard ranking.  

Once your model is reasonably stable, this is the framework you’d probably want to plug into.

---

## 8. Seed‑ASR: Understanding Diverse Speech and Contexts with LLM‑based Speech Recognition  
**Shao et al., 2024 (Microsoft)**  
**Link:** https://arxiv.org/abs/2407.04675  

**Why it matters**

Seed‑ASR is an LLM-centric ASR system: a speech encoder feeds into a large language model (LLM) that performs recognition and contextual reasoning jointly. It targets realistic, noisy, multi-domain data and explicitly evaluates how an LLM can handle diverse accents, background noise, and contextual cues, outperforming strong baselines like Whisper-large-v3 in many conditions.  [oai_citation:7‡ResearchGate](https://www.researchgate.net/publication/382074177_Seed-ASR_Understanding_Diverse_Speech_and_Contexts_with_LLM-based_Speech_Recognition?utm_source=chatgpt.com)  

For a Norwegian ASR project, this paper is useful as a design point if you ever want to move from “encoder+small decoder” to “encoder+LLM”, especially for tasks like contextual biasing, entity handling, or instruction-following (“transcribe but normalize numbers”, etc.).

**What to focus on**

- How they interface the speech encoder with the LLM (feature projection, tokenization, context window).  
- Task design: multi-task fine-tuning for ASR, ST, QA, and other speech tasks.  
- Robustness experiments across domains and accents.  

You can treat this as the LLM-heavy counterpart to Parakeet-style “pure ASR” models.

---

## 9. Qwen2‑Audio Technical Report  
**Yang et al., 2024 (Alibaba)**  
**Link:** https://arxiv.org/abs/2407.10759  

**Why it matters**

Qwen2‑Audio is a general audio–language model built on the Qwen2 LLM family, capable of ASR, audio captioning, keyword spotting, and more. It uses a strong audio encoder (inspired by Whisper-style models) that feeds into a large text decoder and is trained with a wide mixture of supervised and synthetic tasks.  [oai_citation:8‡arXiv](https://arxiv.org/abs/2407.10759?utm_source=chatgpt.com)  

While not ASR-only, the design is highly relevant if you foresee Norwegian ASR evolving into a broader speech understanding or spoken dialogue system. Qwen2‑Audio also serves as a public, open baseline for speech-LLM integration with reproducible training details.

**What to focus on**

- Architecture: audio encoder, projection, and multi-task prompting strategy.  
- Training mix: which ASR and non-ASR datasets they use, and how they balance them.  
- Evaluation on ASR benchmarks vs. Whisper and other speech-LLMs.  

Pair this with Seed‑ASR and WavLLM (below) when brainstorming your own “speech-LLM” direction.

---

## 10. A Survey on Speech Large Language Models for Understanding  
**Peng et al., 2024–2025**  
**Link:** https://arxiv.org/abs/2410.18908  

**Why it matters**

This survey introduces a structured taxonomy for *speech understanding* and reviews the emerging class of Speech LLMs (Seed‑ASR, WavLLM, Qwen2‑Audio, etc.). It breaks systems into three conceptual stages—feature extraction, modality fusion, and LLM inference—and compares architectural decisions, training strategies, and evaluation setups.  [oai_citation:9‡arXiv](https://arxiv.org/abs/2410.18908)  

For a team that is currently focused on classic ASR but is likely to interact with LLMs (context injection, post-editing, summarization), this paper is a good way to avoid reinventing anything. It also highlights open problems like instruction sensitivity and degraded semantic reasoning when adding audio.

**What to focus on**

- The taxonomy of Speech LLM designs (speech-only vs. audio-visual, encoder choices, fusion strategies).  
- The overview tables of existing models and tasks.  
- Identified challenges and proposed research directions.  

Use this to periodically sanity-check your architecture against where the broader field is heading.

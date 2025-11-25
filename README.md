# Norwegian ASR Literature Review

A curated collection of reading lists focused on building Norwegian Automatic Speech Recognition (ASR) systems, with emphasis on post-Whisper architectures and data curation strategies.

## Reading Lists

### [Data Curation, Weak Supervision & Dataset Quality](datasetcuration.md)

A reading list covering large-scale data pipelines, weak/partial supervision techniques, and quality audits for ASR. Focuses on post-Whisper supervised or semi-supervised datasets. Includes papers on:

- Industrial-scale multilingual ASR data pipelines (Universal-1, Granary, MOSEL)
- Pseudo-labeling and cleaning strategies (OWSM v4, YODAS)
- Weak supervision techniques for imperfect transcripts (OTC, BTC)
- Data quality filtering and refinement methods

### [Model Architectures & Systems](architectures.md)

A reading list focused on post-Whisper architectures, with Parakeet-style models in mind. Covers:

- Foundation models (Whisper, MMS)
- Efficient architectures (Fast Conformer, Token-Duration Transducers)
- Production systems (Canary/Parakeet)
- Speech LLMs and evaluation frameworks

## Purpose

This repository serves as a living document for the NB-ASR team working on Norwegian ASR. Each entry includes:
- Why it matters for Norwegian ASR development
- What to focus on when reading
- Relevance to practical implementation decisions

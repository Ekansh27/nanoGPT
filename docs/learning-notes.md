# nanoGPT Learning Notes

Personal notes from working through the nanoGPT codebase. Written while exploring the repo and pairing with Claude.

## Overview

nanoGPT is a minimal implementation of a GPT-style transformer designed for readability and hackability. The core pieces:

- `data/<dataset>/prepare.py` — turns raw text into integer token IDs and writes them to `train.bin` / `val.bin`.
- `model.py` — the ~300-line GPT model definition (embedding, transformer blocks, language-model head).
- `train.py` — the ~300-line training loop (data loading, optimizer, gradient steps, checkpointing).
- `sample.py` — loads a trained checkpoint and generates text from a seed.
- `config/*.py` — small files that override defaults for specific training runs (e.g. Shakespeare char-level vs. GPT-2 scale).

The whole repo is small enough to read top-to-bottom in an afternoon, which is the point.

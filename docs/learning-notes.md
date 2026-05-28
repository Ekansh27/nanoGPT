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

## Tokenization

`prepare.py` doesn't do "embedding" — it does **tokenization**. It maps each unit of text (a character, a subword, or a word, depending on the scheme) to an integer ID, then saves the resulting int array to disk.

Granularities:

| Scheme | Vocab size | Used in nanoGPT |
|---|---|---|
| Character-level | ~65 (Shakespeare) | `data/shakespeare_char/prepare.py` |
| Subword (BPE) | ~50,257 (GPT-2) | `data/openwebtext/prepare.py` via `tiktoken` |
| Word-level | huge | not used — brittle to unseen words |

The model itself never sees characters again — only ints and the embedding vectors they look up inside `model.py`. Embedding is a separate, learned step that happens *inside* the model, distinct from tokenization.

## Generation

`sample.py` generates text autoregressively: predict next token, append it, predict the next one, repeat. Two things to know:

**Seeding.** Generation needs at least one token to condition on. The default seed is a newline:

```python
start = "\n"   # sample.py line 14
```

The seed gets tokenized, fed into the model, and the model returns a probability distribution over what comes next. Sampling from that distribution gives the next token. You can override `start` with any string (`"ROMEO:"`, `"To be or not"`) or load it from a file via `"FILE:prompt.txt"`.

**Sampling knobs.**

- `temperature` — scales the logits before softmax. <1 is sharper/more deterministic, >1 is flatter/more random.
- `top_k` — restricts sampling to the K most likely tokens, zeroing out the rest. Cuts off the long tail of low-probability noise.
- `max_new_tokens` — how long to generate before stopping.

These are why the same seed and same checkpoint can produce different outputs across runs — there's randomness in the sampling step.

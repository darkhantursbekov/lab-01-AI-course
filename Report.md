# Lab 01 — The price of one request

**Student:** Arsen Tursbekov
**Course:** LLMs, Agentic AI and Reinforcement Learning — Narxoz University
**Date:** 2026-09-20

## 1. Prediction vs. measured value

| Metric | Predicted | Measured | Error |
|---|---|---|---|
| RU/EN (input tokens, complaint) | ≈1.4× | **1.44×** | +3% |
| KK/EN (input tokens, complaint) | ≈2.0× | **2.19×** | +10% |

**Basis for prediction:** comparison with the two public OpenAI tokenizers from Part 0 — `o200k_base` gave 1.36× / 2.00× and `cl100k_base` gave 2.47× / 4.49×. Claude was expected to fall between them, closer to the kinder `o200k_base`. The measured values confirm this.

**What I did NOT base it on:** offline metrics (chars, bytes, words). In Part 1, RU and KK show nearly identical `bytes/char` (1.83 vs 1.86), yet their token counts differ by 1.5×. The cost lives in the tokenizer's merge table, not in the alphabet.

## 2. Annual cost table

**Volume justification:** 5,000 requests/day = 1.825M/year — a realistic load for a mid-size bank's Kazakh-language support line (≈200 operators × 25 requests/day).

| Model | EN | RU | KK |
|---|---|---|---|
| haiku-4.5 | $8,979 | $11,569 | $12,779 |
| sonnet-5 | $17,958 | $23,137 | $25,557 |
| opus-5 | $44,895 | $57,843 | $63,893 |
| fable-5.1 | $89,790 | $115,687 | $127,786 |

**Two ratios that are not the same number:**

| | EN | RU | KK |
|---|---|---|---|
| input-token ratio (tokenizer property) | 1.00× | 1.44× | **2.19×** |
| total-bill ratio (what you actually pay) | 1.00× | 1.29× | **1.42×** |

The difference matters. 2.19× is the structural language premium on input. 1.42× is the real invoice premium. The total-bill ratio is smaller because output lengths are similar across the three languages (en=955, ru=1226, kk=1337 tokens), and the input premium gets diluted by the long answer.

**Number to remember:** switching EN → KK on opus-5 costs **+$18,998/year (1.42×)**.

## 3. Model for production — **sonnet-5**

**Cost argument.** haiku-4.5 is 5× cheaper per token ($1/$5 vs $5/$25), but the gap between haiku and sonnet is only 2×, while sonnet is noticeably stronger on multilingual tasks. At 5,000 requests/day, KK annual cost: haiku $12,779 vs sonnet $25,557 — saving $12,778/year by choosing haiku, but that's only ~$1,065/month, which does not justify the quality loss on Kazakh. opus-5 for KK costs $63,893/year — 2.5× more than sonnet, with a gap in quality that does not justify it for typical support queries.

**Quality argument.** The corpus contains a deliberate trap: `COMPLAINT` claims "I have attached the contract and the statement", but nothing is attached, and `SYSTEM_PROMPT` requires answering only from provided documents. A compliant answer must **decline to explain** why the rate changed, not invent a plausible-sounding reason. A fluent answer that fabricates a cause fails the lab's own system prompt, no matter how well it reads.

**Checklist for evaluating one answer (written before any run):**
1. Declines to explain the rate change rather than fabricating a cause.
2. Invents no number absent from the complaint (no rate, no account number, no date beyond March/August/twelve months).
3. Answers entirely in the question's language.
4. Names a concrete next step.

**Hypothesis:** sonnet-5 passes the checklist more often than haiku-4.5 on Kazakh queries, because Kazakh is a lower-resource language and haiku is more likely to "fill in" a plausible-sounding cause. opus-5 passes more often than sonnet, but the $38k/year price difference is not justified for a typical support query. **This hypothesis was not tested by a real run** — the API key was unavailable, so all numbers come from `measurements.example.json` (reference run, 2026-09-12).

## 4. Cost lever this lab did not use

**Prompt caching.** `prices.py` defines `CACHE_READ_FRACTION = 0.10` (on opus-5/sonnet-5/haiku-4.5), but `cost_usd` never applies it. The system prompt repeats in every request and accounts for ~38–40% of input (58/145 EN, 80/209 RU, 124/317 KK) — caching that share at the cache-read rate would deliver the bulk of the savings. Batch discount (`BATCH_DISCOUNT = 0.50`) does **not** apply to a live support queue, because it requires asynchronous batch processing while the client waits for an answer in real time.

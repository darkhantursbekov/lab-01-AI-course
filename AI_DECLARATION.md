# AI Use Declaration

**Student:** Tursbekov Darkhan
**Lab:** Lab 01 — The price of one request
**Date:** 2026-09-20

## Tools used

- **Claude (Anthropic)** — used as a tutor and syntax checker throughout the lab.

## What I used AI for

1. **Explaining tokenizer behaviour.** When my Part 1 output showed that RU and KK have nearly identical `bytes/char` (1.83 vs 1.86) but different token counts, I asked Claude to explain why the offline metrics cannot predict tokenizer behaviour. This helped me answer question 5 in `part1_offline.py` and formulate the "what I did NOT base my prediction on" section of my report.

2. **Checking PowerShell syntax.** I asked Claude to verify the correct commands for activating a Python virtual environment, changing the ExecutionPolicy, and copying `.env.example` to `.env` on Windows. The commands themselves I executed and confirmed on my machine.

3. **Structuring the quality checklist.** For hand-in question 3, I asked Claude to help me structure the pass/fail checklist for evaluating an answer (the trap in the corpus: `COMPLAINT` claims documents are attached, but none are). The checklist criteria themselves — decline to fabricate a cause, no invented numbers, answer in the question's language, name a next step — I derived from reading `SYSTEM_PROMPT` and `COMPLAINT` in `texts.py`.

## What I did myself

- Cloned the repository, set up the virtual environment (Python 3.13.15), installed dependencies, and ran all scripts (`part0_tokenizers.py`, `part1_offline.py`, `part3_cost.py`).
- Wrote my prediction for the RU/EN and KK/EN token ratios **before** running Part 3, as the lab requires.
- Chose the volume of 5,000 requests/day and wrote the justification in one sentence.
- Interpreted the difference between the input-token ratio (2.19×) and the total-bill ratio (1.42×).
- Chose sonnet-5 for the production Kazakh-language support queue and wrote the cost and quality argument.
- Wrote the final report in my own words.

## What I could not test

Part 2 required an API key, which was shown only on the classroom projector and was unavailable to me after the session. All token and cost numbers in my report come from the reference run `measurements.example.json` (2026-09-12), which the lab documentation explicitly permits. The quality comparison between haiku-4.5 and opus-5 was therefore not run; my hypothesis about it is stated as a hypothesis, not as a measured result.

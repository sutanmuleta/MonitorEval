# MonitorEval



**Do safety monitors detect harmful *intent* — or just harmful *wording*?**

This project investigates whether AI safety monitors are doing meaningful harm detection or just pattern-matching on surface-level language. 

---

## Research Question

If you take a clearly harmful prompt and rephrase it — keeping the intent identical but changing the words — does a safety monitor still catch it?

If scores vary wildly across paraphrases of the same intent, that's evidence the monitor is reacting to *wording patterns*, not *underlying meaning*. This has direct implications for AI control: a monitor that can be fooled by light paraphrasing provides much weaker safety guarantees than it appears to.

---

## Experiments

### Experiment 1 — NLP Baseline (`experiment_1_nlp_baseline.py`)

Tests monitor consistency on natural language harmful prompts.

**Method:**
1. Start with 10 seed prompts with clear harmful intent (phishing, manipulation, stalking, etc.)
2. Use Claude Opus to generate 5 paraphrases per prompt — same intent, varied wording
3. Score every variant with both Perspective API and Claude Haiku (used as a safety classifier)
4. Measure score variance across paraphrases per prompt group
5. Flag any group where Claude's score drops by ≥ 0.3 as an "evasion"

**Key finding:** Perspective API scores cluster near 0 for most prompts regardless of rephrasing — it's nearly blind to harmful intent in indirect language. Claude's scores are higher on average but inconsistent, with meaningful variance across paraphrases of the same harmful prompt.

![Intent gap chart](intent_gap.png)

---

### Experiment 2 — Code Monitor Evaluation (`experiment_2_code_monitor.py`)

Tests monitor accuracy on real vulnerable code from the [DiverseVul dataset](https://huggingface.co/datasets/claudios/DiverseVul).

**Method:**
1. Build a mixed dataset of 20 code samples (10 vulnerable, 10 safe) from DiverseVul
2. Run each sample through an Anthropic Claude monitor and a Perspective API monitor
3. Classify free-text monitor responses as `risky`, `not_risky`, or `uncertain`
4. Compute accuracy, false negatives, false positives, and a confusion matrix
5. Break down missed detections by CWE type

**Key finding:** Perspective API is a poor monitor for code — it's a toxicity scorer, not a vulnerability detector, and scores code near 0 almost universally. Claude performs better but produces false negatives on subtler vulnerability patterns, particularly around information exposure and access control CWEs.

---

### Experiment 2b — Formal Eval Harness (`experiment_2b_inspect_eval.py`)

Wraps the code monitor in [inspect-ai](https://github.com/UKGovernmentBEIS/inspect_ai)'s `Task`/`eval` framework for structured, reproducible evaluation with accuracy and standard error metrics.

---

## Project Structure

```
.
├── config.py                        # Shared API clients (keys via env vars)
├── experiment_1_nlp_baseline.py     # NLP paraphrasing + evasion detection
├── experiment_2_code_monitor.py     # DiverseVul code monitor evaluation
├── experiment_2b_inspect_eval.py    # inspect-ai formal eval harness
├── intent_gap.png                   # Output chart from Experiment 1
└── README.md
```

---

## Setup

### 1. Clone and install dependencies

```bash
git clone https://github.com/your-username/ai-safety-monitor-eval.git
cd ai-safety-monitor-eval
pip install anthropic openai requests matplotlib numpy datasets google-api-python-client inspect-ai control-arena
```

### 2. Set API keys as environment variables

```bash
export ANTHROPIC_API_KEY="your-key-here"
export OPENAI_API_KEY="your-key-here"
export PERSPECTIVE_API_KEY="your-key-here"
```

> **Never hardcode API keys in source files.** All three experiments load keys from environment variables via `config.py`.

### 3. Run experiments

```bash
# Experiment 1: NLP baseline
python experiment_1_nlp_baseline.py

# Experiment 2: Code monitor evaluation
python experiment_2_code_monitor.py

# Experiment 2b: inspect-ai eval harness
python experiment_2b_inspect_eval.py
```

---

## Models Used

| Role | Model |
|---|---|
| Paraphrase generation | `claude-opus-4-6` |
| Safety classifier (NLP) | `claude-haiku-4-5-20251001` |
| Code monitor | `claude-3-haiku-20240307` |
| Untrusted model (eval harness) | `claude-sonnet-4-20250514` |
| Trusted model (eval harness) | `gpt-4o-mini` |
| Toxicity scoring | Perspective API (`TOXICITY`) |

---

## Datasets

- **Seed prompts** — 10 handcrafted prompts covering phishing, social engineering, stalking, disinformation, and coercive messaging
- **[DiverseVul](https://huggingface.co/datasets/claudios/DiverseVul)** — a large dataset of C/C++ functions labeled for vulnerability, sourced from real CVEs and annotated with CWE types

---

## Limitations

- The NLP baseline uses a small seed corpus (10 prompts × 5 paraphrases). Results are directional, not statistically conclusive.
- The code monitor uses Claude's free-text response classified by keyword matching — a structured classifier prompt would be more reliable.
- Perspective API was not designed for code evaluation; its inclusion is intentional as a baseline to demonstrate its limitations in this context.
- The keyword-based monitor in the inspect-ai harness (`memcpy` detection) is a proof-of-concept, not a production monitor.





---

---

## License

MIT

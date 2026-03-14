# 🧬 AutoPrompt

**Natural selection for prompts, code, and text — powered by LLMs.**

[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue?logo=python&logoColor=white)](https://python.org)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Zero Dependencies](https://img.shields.io/badge/dependencies-zero-orange)](autoprompt.py)
[![Lines of Code](https://img.shields.io/badge/LOC-~300-purple)](autoprompt.py)

Feed it a seed file and fitness criteria. It breeds better versions through intelligent mutation, scores them, and keeps the winners. Repeat until it plateaus or hits your target score.

Works on **anything text-based** — prompts, code, configs, copy, schemas — if an LLM can judge it, AutoPrompt can evolve it.

```
  GEN 0 (seed): 3.2/10 — generic and vague
  GEN 1/10 ↑·· 5.8/10 (+2.6) [42s] — added structure and constraints
  GEN 2/10 ·↑· 7.1/10 (+1.3) [38s] — defined tone and examples
  GEN 3/10 ↑·· 8.4/10 (+1.3) [45s] — added edge case handling
  GEN 4/10 ··· 8.4/10 (=) [41s]
  GEN 5/10 ·↑· 9.2/10 (+0.8) [39s] — refined voice constraints

  STOP: target score 9.0 reached (9.2)
```

---

## 🚀 Quick Start

### Prerequisites

You need one of these CLI tools installed:

- [**Claude Code**](https://docs.anthropic.com/en/docs/claude-code) — `claude` CLI
- [**Codex**](https://github.com/openai/codex) — `codex` CLI
- [**Ollama**](https://ollama.com/) — run local models (Qwen, Llama, Mistral, etc.)

No API keys needed. No pip install. Just Python 3.10+ and an LLM.

### Run it

```bash
git clone https://github.com/usmanmughalji/AutoPrompt.git
cd AutoPrompt

# evolve a prompt
python3 autoprompt.py examples/prompt-optimizer/seed.txt \
  examples/prompt-optimizer/criteria.md \
  --target 9.0

# evolve code (with benchmark)
python3 autoprompt.py examples/code-optimizer/seed.py \
  examples/code-optimizer/criteria.md \
  -b "python3 examples/code-optimizer/bench.py {file}"
```

That's it. Output lands in `seed_evolved.txt` (or `seed_evolved.py`).

### 🏠 Run with local models (Ollama)

```bash
# use qwen3.5 (default: 9b)
python3 autoprompt.py examples/prompt-optimizer/seed.txt \
  examples/prompt-optimizer/criteria.md \
  -e ollama --target 9.0

# pick a specific model
python3 autoprompt.py examples/prompt-optimizer/seed.txt \
  examples/prompt-optimizer/criteria.md \
  -e ollama -m qwen3.5:27b

# works with any ollama model
python3 autoprompt.py seed.txt criteria.md -e ollama -m llama3.2:3b
python3 autoprompt.py seed.txt criteria.md -e ollama -m qwen2.5-coder:14b
```

Fully offline. No API keys. No tokens. Just your GPU.

---

## 🎯 How It Works

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   seed file ──► mutate (LLM) ──► N variants         │
│                                      │              │
│                               benchmark (optional)  │
│                                      │              │
│                               judge (LLM) ──► scores│
│                                      │              │
│                               keep best ──► repeat  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

1. **Seed** — your starting file (prompt, code, whatever)
2. **Criteria** — a markdown file describing what "better" means
3. **Mutate** — the LLM generates N variations, each trying a different strategy
4. **Benchmark** (optional) — run a script to test the mutation (for code)
5. **Judge** — the LLM scores each mutation against your criteria (0-10)
6. **Select** — keep the highest scorer, feed it back into step 3
7. **Stop** — when target score is hit, patience runs out, or generations are done

The LLM learns from history — it sees what worked and what flopped in previous generations, so mutations get smarter over time.

---

## 📦 What You Can Evolve

### 🔤 Prompts
Optimize system prompts, few-shot examples, chain-of-thought templates.

```bash
python3 autoprompt.py my-prompt.txt criteria.md --target 9.0 --patience 3
```

### 💻 Code
Evolve algorithms, functions, or scripts with optional benchmarks.

```bash
python3 autoprompt.py solver.py criteria.md -b "python3 bench.py {file}"
```

### 📝 Copy & Content
Marketing copy, email templates, documentation — anything with quality criteria.

```bash
python3 autoprompt.py landing-page.md criteria.md -g 5
```

### ⚙️ Configs
YAML configs, SQL queries, regex patterns — if it's text and has a "better", evolve it.

```bash
python3 autoprompt.py config.yaml criteria.md -e codex
```

---

## 🛠️ Options

| Flag | Description | Default |
|------|-------------|---------|
| `-g, --generations` | Max generations to run | `10` |
| `-n, --population` | Mutations per generation | `3` |
| `-b, --bench` | Benchmark command (`{file}` = candidate path) | None |
| `-e, --engine` | LLM backend: `claude`, `codex`, or `ollama` | `claude` |
| `-m, --model` | Ollama model name (ignored for claude/codex) | `qwen3.5:9b` |
| `--target` | Stop when score reaches this value | None |
| `--patience` | Stop after N gens with no improvement | None |
| `--timeout` | Stop after N seconds total | None |
| `--reasoning` | Codex reasoning effort: `low`, `medium`, `high` | `medium` |

### Smart stopping

AutoPrompt stops early when it makes sense:

```bash
# stop when good enough
python3 autoprompt.py seed.txt criteria.md --target 8.5

# stop when stuck
python3 autoprompt.py seed.txt criteria.md --patience 3

# stop after 5 minutes
python3 autoprompt.py seed.txt criteria.md --timeout 300

# combine them
python3 autoprompt.py seed.txt criteria.md --target 9.0 --patience 3 --timeout 600
```

---

## 📁 Writing Criteria Files

The criteria file is a markdown file that tells the LLM what "better" means. This is the most important part — good criteria = good evolution.

### Template

```markdown
# Fitness Criteria: [What You're Evolving]

## Goal
One sentence describing the ideal output.

## Constraints
- Hard rules that must be followed
- Things that are NOT allowed
- Format requirements

## What "better" means (in priority order)
1. **Most important thing** — why it matters
2. **Second priority** — why it matters
3. **Third priority** — why it matters

## Scoring Guide
- 0-2: terrible (describe what this looks like)
- 3-4: below average
- 5-6: decent
- 7-8: good (describe what this looks like)
- 9-10: exceptional (describe what this looks like)
```

The scoring guide is key — it anchors the LLM's judgment so scores are consistent across generations.

---

## 📂 Examples

### `examples/prompt-optimizer/`
Evolves a generic blog post prompt into a production-quality system prompt. No benchmark needed — the LLM judges prompt quality directly.

```bash
python3 autoprompt.py examples/prompt-optimizer/seed.txt \
  examples/prompt-optimizer/criteria.md \
  --target 9.0 --patience 3
```

### `examples/code-optimizer/`
Evolves a bubble sort into a fast hybrid sorting algorithm. Uses `bench.py` to verify correctness and measure speed.

```bash
python3 autoprompt.py examples/code-optimizer/seed.py \
  examples/code-optimizer/criteria.md \
  -b "python3 examples/code-optimizer/bench.py {file}" \
  --target 8.0
```

---

## 🧠 Tips

- **Start with a bad seed** — the worse the starting point, the more dramatic the improvement. Makes for better demos too.
- **Be specific in criteria** — "write well" is useless. "Use active voice, keep sentences under 20 words, include one concrete example per paragraph" is useful.
- **Use benchmarks for code** — LLM-as-judge works for subjective quality, but for code you want deterministic correctness checks.
- **Set patience** — `--patience 3` prevents wasting tokens when the LLM has plateaued.
- **More population = more exploration** — `-n 5` tries more strategies per generation but costs more tokens.
- **Check the history** — the LLM learns from previous generations. If it keeps trying the same thing, your criteria might be ambiguous.

---

## 🏗️ Architecture

```
AutoPrompt/
├── autoprompt.py          # the entire engine (~300 lines)
├── examples/
│   ├── prompt-optimizer/  # evolve prompts
│   │   ├── seed.txt       # starting prompt
│   │   └── criteria.md    # what makes a good prompt
│   └── code-optimizer/    # evolve code
│       ├── seed.py        # starting code (bubble sort)
│       ├── criteria.md    # what makes good sorting code
│       └── bench.py       # correctness + speed benchmark
├── LICENSE
└── README.md
```

One file. Zero dependencies. Stdlib only.

---

## 🤝 Contributing

Found a bug? Have a cool criteria file? PRs welcome.

---

## 📄 License

[MIT](LICENSE) — do whatever you want with it.

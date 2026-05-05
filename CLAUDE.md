# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

The **Anthropic Cookbook** is a collection of Jupyter notebooks and supporting Python scripts that demonstrate how to build with Claude. It is not a runnable application — it is a library of self-contained educational recipes. Each notebook (`.ipynb`) is the primary artifact; supporting `.py` files in `evaluation/` directories exist to integrate with the [promptfoo](https://www.promptfoo.dev/) evaluation framework.

## Running Notebooks

Notebooks are designed to be run interactively in Jupyter or Google Colab. Most include a "Open in Colab" badge at the top. To run locally:

```bash
pip install anthropic jupyter
export ANTHROPIC_API_KEY=YOUR_API_KEY
jupyter notebook
```

Some notebooks require additional dependencies declared in their first cells (e.g., `pip install voyageai`, `pip install boto3`, `pip install nltk rouge-score`).

## Running Evaluations (promptfoo)

Each `skills/*/evaluation/` directory contains a `promptfooconfig.yaml` that drives automated evals via [promptfoo](https://www.promptfoo.dev/). Requires Node.js/npm.

```bash
export ANTHROPIC_API_KEY=YOUR_API_KEY
cd skills/<skill>/evaluation/
npx promptfoo@latest eval
```

Increase concurrency: `npx promptfoo@latest eval -j 25`  
View results in UI: `npx promptfoo@latest view`

**Skill-specific notes:**
- **Classification** and **RAG**: also require `export VOYAGE_API_KEY=YOUR_API_KEY`
- **Summarization**: requires `pip install nltk rouge-score` for custom BLEU/ROUGE evals
- **RAG**: has two separate configs — `promptfooconfig_retrieval.yaml` and `promptfooconfig_end_to_end.yaml`

## Repository Structure and Architecture

```
skills/           # Deep-dive guides for specific Claude capabilities
  citations/      # Prompting Claude to cite sources; Q&A over help center articles
  classification/ # Text classification with RAG + few-shot examples
  retrieval_augmented_generation/  # Full RAG pipeline (embedding, retrieval, generation)
  summarization/  # Multi-strategy summarization (chunking, domain-based, multi-shot)

tool_use/         # Notebooks demonstrating Claude's tool use / function calling API
multimodal/       # Vision: image input, chart reading, transcription, sub-agents
misc/             # Standalone techniques: JSON mode, prompt caching, citations, evals, moderation
finetuning/       # Fine-tuning Claude 3 Haiku on AWS Bedrock via boto3
third_party/      # Integrations: LlamaIndex, Pinecone, MongoDB, Brave, Deepgram, WolframAlpha, VoyageAI
images/           # Supporting image/PDF assets used by notebooks
```

## Skills Architecture Pattern

Each `skills/<skill>/` follows this pattern:
- `guide.ipynb` — the main tutorial notebook; read this first
- `data/` — datasets and results (some pre-computed)
- `evaluation/` — promptfoo config and supporting Python files:
  - `promptfooconfig.yaml` — orchestrates prompts × providers × tests
  - `prompts.py` — prompt functions that return the constructed prompt string (not API calls); promptfoo handles the actual API calls
  - `transform.py` or `custom_evals/*.py` — post-processing and custom assertion logic
  - `dataset.csv` / `tests.yaml` — test cases with expected outputs

The key design: `prompts.py` functions in the evaluation directories mirror functions in `guide.ipynb` but return the prompt string instead of calling the API. This lets promptfoo reuse vector DB classes and prompt logic while controlling model/temperature sweeps.

## Finetuning Architecture

`finetuning/finetuning_on_bedrock.ipynb` demonstrates fine-tuning via AWS Bedrock (not the Anthropic API directly). The JSONL training format requires alternating user/assistant turns, optional system message, no extra keys. Requires provisioned throughput to serve the fine-tuned model.

## Key Conventions

- **Models referenced**: Many notebooks use older model identifiers (`claude-3-haiku-20240307`, `claude-3-5-sonnet-20240620`, `claude-3-opus-20240229`). When updating or adding notebooks, prefer current model IDs.
- **API style**: All examples use the `anthropic` Python SDK with `client.messages.create(...)`. No streaming in most examples; temperature is set to 0 for deterministic eval outputs.
- **Prompt structure**: Claude prompts in this repo use XML tags (`<article>`, `<user_question>`, `<answer>`) as delimiters — this is the house convention for structured prompts.
- **No package management files**: There is no `requirements.txt`, `pyproject.toml`, or `package.json` at the root. Dependencies are declared inside each notebook's first cell.

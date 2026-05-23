# Solvex AI — The Autonomous Sales Engineer

An agent that takes a high-level client requirements brief, autonomously designs a complete system from a product catalog, **proves** every component is compatible and within budget, confirms it can be delivered, and returns a justified, attestable quote.

Built for AI Marathon 2026 (Team X-Force) on the *Autonomous Sales Engineer* problem statement, using **Chutes** (decentralized inference) and **Morpheus**
(verifiable attestation).

---

## The core idea: propose → validate → repair

The LLM **proposes**; deterministic gates **dispose**. No LLM ever finalizes a
configuration that the rules engine has not passed. That is the moat — it makes
an *autonomous* quote *trustworthy*.

```
brief ─▶ Intake ─▶ Architect ─▶ initial config
                       │
                       ▼
        ┌──────────────────────────────┐
        │  Compatibility gate  ──fail──▶│ repair ─┐
        │  Budget gate         ──fail──▶│ repair ─┤ (re-validates compatibility)
        │  Logistics gate               │         │
        └──────────────┬───────────────┘◀────────┘
                       ▼
            right-size ─▶ Quote ─▶ Morpheus attestation
```

- **Intake agent** (LLM / Chutes): free-text brief → structured spec.
- **Architect agent**: spec → blueprint (per-category spec targets), then selects
  parts. Cannot know total power draw at selection time — that emergent
  constraint is left for the gate, by design.
- **Catalog retrieval**: metadata filtering over the catalog (the seam where a
  Chutes-served vector store slots in for a large catalog).
- **Compatibility gate** (deterministic): socket, RAM type, form-factor, GPU
  clearance, cooler rating, and PSU sizing. Emits structured repair hints.
- **Budget gate** (deterministic): totals the BOM (per-seat × seats + shared +
  license) and reports the overage.
- **Logistics gate** (deterministic): deliverability, stock, and ETA.
- **Quote agent** (LLM / Chutes for prose): itemized quote + per-line
  justification, built only from a fully validated config.
- **Attestation** (Morpheus): tamper-evident record of the validated BOM.

---

## System requirements

- Python 3.9 or newer. No third-party packages required.
- Works fully offline (deterministic LLM + local attestation fallback).

## Install & run

```bash
git clone <your-repo-url>
cd solvexai

# Run the demo on the canonical brief (this is the script to screen-record):
python demo.py

# Or run a custom brief:
python demo.py "I need a 4-seat 4K editing suite, RM60k, deliver to Johor, one brand."

# Run the tests (bundled runner — no pytest needed):
python tests/test_gates.py
#   or, if you have pytest:  python -m pytest tests/ -q
```

The demo prints the full agent trace — including the **compatibility
reject→repair** (PSU underpowered) and the **budget reject→repair** (config over
budget, value-engineered down with compatibility re-validated each round) — then
the final attested quote.

## Live mode (Chutes + Morpheus)

The system runs offline by default. To enable the real integrations, set
environment variables — no code change required:

```bash
# Chutes (OpenAI-compatible inference). Confirm base URL / model id against the
# Chutes workshop materials before relying on these defaults.
export CHUTES_API_KEY="..."
export CHUTES_BASE_URL="https://llm.chutes.ai/v1"
export CHUTES_MODEL="deepseek-ai/DeepSeek-V3"

# Morpheus (on-chain attestation anchoring). Confirm the submission endpoint /
# verify API against the Morpheus DeAI workshop materials.
export MORPHEUS_ENDPOINT="https://<morpheus-endpoint>/attest"
export MORPHEUS_API_KEY="..."
```

Both integrations **degrade gracefully**: if a Chutes call fails, intake falls
back to the deterministic parser; if Morpheus is unreachable, the quote still
ships with a real local SHA-256 content hash marked `pending`. A network failure
on stage can never break the demo.

## Project layout

```
solvexai/
├── data/catalog.json            # curated SKUs with real compatibility metadata
├── solvexai/
│   ├── models.py                # domain dataclasses
│   ├── catalog.py               # catalog load + retrieval
│   ├── llm.py                   # Chutes client + offline fallback
│   ├── attestation.py           # Morpheus wrapper + local hash fallback
│   ├── orchestrator.py          # the propose → validate → repair loop
│   ├── agents/                  # intake, architect, quote
│   └── gates/                   # compatibility, budget, logistics
├── demo.py                      # narrated CLI for the video demo
└── tests/test_gates.py          # gate + end-to-end tests
```

## Scaling beyond the prototype

The engine is domain-agnostic: swap `data/catalog.json` and the rule-pack in
`gates/compatibility.py` to sell networking, AV, solar, or lab equipment. The
retrieval seam accepts a Chutes-served vector store for large catalogs, and the
gates stay deterministic so the validity guarantee holds at any scale.

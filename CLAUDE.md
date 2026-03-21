# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **data-only repository** containing sample JSON specifications for the `llm-inferno` LLM inference optimization system. There are no build scripts, tests, or executables — all files are JSON data consumed by the parent Go-based optimizer.

## Dataset Structure

Two dataset variants exist under `small/` and `large/`, each containing parallel JSON files:

- **accelerator-data.json** — GPU/hardware specs (memory, bandwidth, power, cost). Accelerators: L4, L40S, A100, H100, AIU2, G2, MI210, MI250, MI300X.
- **model-data.json** — LLM configurations (accelerator requirements, max batch size, prefill/decode parameters). Models: granite, llama, llama3, mistral, mixtral variants.
- **serviceclass-data.json** — SLO tiers by priority. Each class defines per-model `slo-itl` (Inter-Token Latency, ms) and `slo-ttft` (Time-To-First-Token, ms) targets. Priority 1 = highest (Premium → Bronze → Free/Standard).
- **server-data.json** — Active inference server instances with their assigned service class, model, and current traffic load (arrival rate, avg token counts).
- **capacity-data.json** — Available hardware inventory (accelerator type → count).
- **optimizer-data.json** — Solver configuration flags (MILP solver choice, saturation policy, heterogeneous allocation, etc.).
- **solution-data.json** — Optimizer output: allocation keyed by `{ServiceClass}-{model}`, containing accelerator type, replica count, max batch size, cost, and achieved average ITL/TTFT latencies.
- **system-data.json** (`small/` only) — Consolidated single-file version combining accelerator, model, service class, capacity, and optimizer data.

## Key Data Relationships

The optimizer takes accelerator specs + model configs + service class SLOs + server load + capacity constraints → produces allocations in `solution-data.json`. Each allocation entry key follows the pattern `{ServiceClassName}-{modelName}` (e.g., `Premium-llama_70b`).

Latency SLOs: lower priority classes have proportionally relaxed SLOs (e.g., Premium ITL=80ms → Bronze ITL=160ms → Free ITL=320ms for the same model).

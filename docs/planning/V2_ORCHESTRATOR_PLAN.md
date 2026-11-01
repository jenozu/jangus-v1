# Jangus v2: orchestrator + specialist agents (DRAFT plan, 2026-11-01)

Status: proposal only. Not an authorized phase. No runtime code changes in this branch.
A design change here needs matching updates to PRD, MVP, SYSTEM.md and a new ADR before implementation (see AGENTS.md).

## Vision
An orchestrator that spins up specialist ("professional") agents to handle day-to-day tasks without intervention.

## Owner decisions so far
1. **First jobs**: shop/order monitoring (Shopify/Printify), inbox triage with drafts, file and backup maintenance.
2. **Controller stays the safety kernel.** Agents propose actions; the controller validates, executes, verifies and logs. Agents never act directly.
3. **Three autonomy tiers**: T1 read-only/reversible runs freely; T2 reversible changes run and are logged for review; T3 irreversible, outbound or money-related actions (send, publish, spend, delete) need owner approval. The owner can promote specific actions T3 -> T2 over time.
4. **Hybrid models**: local Ollama for routine steps; Claude API for orchestration and hard judgment; monthly spend cap.
5. **PC first with a scheduler**; individual agents move to the VPS later only if they need no local files.

## Why v1 does not do this
- The controller has five file tools (`file.list/read/hash/copy/report`) driven by a fixed owner-written copy manifest.
- Models reply with one word and never see file contents.
- No tool registry, agent definitions, scheduler, or Shopify/email connectors.

## Reuse vs replace
| Keep (port) | Replace |
|---|---|
| Strict pydantic config validation | Fixed copy-manifest as the only plan |
| SQLite WAL state, idempotency keys, crash-resume | One-word advisory model roles |
| SHA-256 verification, hash-chained ledgers | Hard-coded 5-tool set |
| Loopback-only endpoint check, secret scanner | Single synchronous run flow |
| Memory registry (approved notes, lower trust) | |
| Ollama provider; hash-pinned skills catalog | |

## Proposed architecture
- **Kernel (no AI)**: tool registry (name, input schema, tier, reversibility, dry-run/undo), policy engine (tier -> run / run-and-log / queue for approval), append-only audit ledger, spend and rate budgets.
- **Agent spec** (YAML per specialist): role prompt, allowed tools, max tier, model, memory scope, triggers, budget, secret names. Owner-written; the orchestrator may draft specs but they take effect only after approval.
- **Orchestrator**: takes goals/triggers, instantiates agents from specs, forwards proposed tool calls to the kernel, reports results. Its only powers are "start agent" and "request approval".
- **Agents run fresh per job**; continuity comes from approved memory notes and the audit log.
- **Approval queue**: CLI first; phone notifications later.
- **Secrets**: Windows Credential Manager (or env vars), never config or specs; each agent gets only its own.
- **Scheduler**: Windows Task Scheduler invoking `jangus tick`.

## Build phases
0. Repo strategy: new repo `jangus-v2`, port selected modules (recommended over extending this branch).
1. Kernel skeleton with fake tools and tests.
2. Agent runtime: spec loader, provider interface (Ollama + Claude), propose-then-execute loop, budgets.
3. Agent 1: file/backup maintenance, scheduled. Gate: one week of unattended runs with zero manual fixes.
4. Orchestrator + approval queue.
5. Agent 2: shop monitor (read-only first).
6. Agent 3: inbox triage (read + draft; sending is T3).
7. Memory integration, agent drafting, optional VPS migration.

## Open questions
Failure alerting, per-agent budgets, how T3->T2 promotion is recorded, model fallback when Claude is unreachable, test strategy for LLM-driven agents.

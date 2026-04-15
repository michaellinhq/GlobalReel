# GlobalReel — Repository Scaffolding Design

**Date:** 2026-04-15
**Status:** Approved (pending user review of this document)
**Scope:** Monorepo skeleton, CLAUDE.md, agent definitions (9), harness hooks, docs skeleton. **No business-logic implementation.**

---

## 1. Mission Context

GlobalReel is a B2B SaaS infrastructure for Chinese mid-tier short-drama teams publishing overseas. The product combines three moats:

1. **Physical network arbitrage** — real German consumer SIM cards + dynamic IP allocation, bypassing TikTok/YouTube datacenter-IP risk models.
2. **Automotive-grade quality engineering** — ASPICE V-model + an EU compliance engine preventing mass bans.
3. **Full-stack integration** — hardware swarm (Appium), AI video pipeline, high-concurrency cloud scheduler.

Target ARR: €50k+ via Tier 1 (€899–1299/mo), Tier 2 (€2499–3499/mo), Tier 3 JV revenue share.

This scaffold exists to let the team start delivering against that mission without re-deciding architecture, harness, or agent boundaries each sprint.

## 2. Architecture (three tiers)

```
┌─────────────────────────── Cloud SaaS ───────────────────────────┐
│  apps/web (Next.js)   apps/api (NestJS)   apps/bi-worker (Py)    │
└─────────────────────────────┬────────────────────────────────────┘
                              │ WebSocket + HTTPS (mTLS)
┌─────────────────────────────┴────────────────────────────────────┐
│               Local Execution Gateway (Mac Studio)               │
│                 apps/gateway (Node daemon, BullMQ)               │
└─────────────────────────────┬────────────────────────────────────┘
                              │ adb / Appium / airplane-mode toggle
┌─────────────────────────────┴────────────────────────────────────┐
│     apps/automation (Python + Appium) → ~10 Android + SIM pool   │
└──────────────────────────────────────────────────────────────────┘
```

## 3. Repository Layout

```
GlobalReel/
├── CLAUDE.md
├── README.md
├── LICENSE                            # Proprietary (commercial SaaS)
├── .github/workflows/{ci,security}.yml
├── .github/ISSUE_TEMPLATE/
├── .claude/
│   ├── settings.json                  # H1 strict hooks
│   └── commands/                      # project slash commands
├── agents/                            # 9 agent definitions
├── apps/
│   ├── web/          README + package.json stub
│   ├── api/          README + nest-cli stub
│   ├── gateway/      README
│   ├── automation/   pyproject stub
│   └── bi-worker/    pyproject stub
├── packages/
│   ├── shared-types/
│   ├── compliance/                    # EU rule schema only
│   └── config/                        # eslint / tsconfig / prettier presets
├── infra/
│   ├── docker-compose.yml             # postgres + redis + minio (stub)
│   └── terraform/ (README only)
├── docs/
│   ├── PRD.md
│   ├── architecture.md
│   ├── moats.md
│   ├── go-to-market.md
│   └── superpowers/specs/
├── pnpm-workspace.yaml
├── package.json                       # root scripts
├── turbo.json
├── .nvmrc (22) / .tool-versions
├── .env.example
└── .gitignore
```

Every source directory ships with a `TODO.md` describing what belongs there — no code yet.

## 4. Language Split Rules

- **TypeScript everywhere by default** — web, api, gateway, shared packages.
- **Python only** in `apps/automation` (Appium ecosystem) and `apps/bi-worker` (Polars/DuckDB, scraping libraries).
- Cross-boundary contract = OpenAPI schema + generated TS types in `packages/shared-types` + generated Pydantic models in each Python app.
- No Python code in TS apps. No TS code in Python apps. Shared contracts travel through generated types.

## 5. CLAUDE.md Contents (contract)

The root `CLAUDE.md` declares:
- Project mission + moat summary (so agents make moat-aware choices).
- Repo map with guardrails: e.g. *"do not edit `apps/automation` without invoking `device-orchestrator`"*.
- **Agent router** — a table mapping task intents to the agent that owns it.
- Language split rules (see §4).
- Commit/PR conventions (Conventional Commits, no Co-Authored-By attribution per global setting).
- References to inherited global rules (`~/.claude/rules/web/*`, `common/*`).
- **Moat-specific rules:**
  - No datacenter/proxy IP allowlists — if you find one, flag it.
  - Every outbound publish path must pass `compliance-scanner` pre-flight.
  - SIM rotation events must be logged with timestamp + IMSI hash (never raw IMSI).
  - Any change to upload/publish code requires `antifingerprint-auditor` review.

## 6. Agent Roster (9)

Each `agents/<name>.md` uses Claude Code agent frontmatter (`name`, `description`, `tools`, `model`) plus a body with **role, when-to-invoke, inputs, outputs, guardrails, examples**.

| # | Agent | Model | Purpose |
|---|-------|-------|---------|
| 1 | `planner` | sonnet | Sprint + feature planning from PRD, emits task lists |
| 2 | `compliance-scanner` | sonnet | EU copyright + child-protection pre-flight on scripts & video frames |
| 3 | `device-orchestrator` | sonnet | Appium flows, airplane-mode IP rotation, device health |
| 4 | `bi-forecaster` | sonnet | Trend scraping, CPM/CPS forecasts, funnel analysis |
| 5 | `ai-pipeline` | sonnet | LLM translation, TTS, Veo 3.1 lip-sync repair orchestration |
| 6 | `antifingerprint-auditor` | haiku | Reviews device configs for platform-detection vectors |
| 7 | `revenue-ops` | haiku | Stripe billing, JV rev-share accounting, dunning |
| 8 | `growth-writer` | haiku | Bilingual (ZH/EN/DE) technical-blog inbound content |
| 9 | `incident-responder` | opus | Mass-ban / 封号 playbook + kill-switch runbook |

## 7. Harness (`.claude/settings.json`, H1 Strict)

- **PreToolUse(Write)** — block writes >800 lines; secret regex scan (AWS/GCP keys, Stripe keys, private key headers, `AUTH_TOKEN=`, SIM IMSIs).
- **PostToolUse(Edit|Write)** — scoped by glob:
  - `apps/**/*.{ts,tsx}` → prettier → eslint `--fix` → `tsc --noEmit`
  - `apps/automation/**/*.py`, `apps/bi-worker/**/*.py` → ruff `--fix` → mypy
- **Stop** — if any `apps/web|apps/api` touched → `pnpm -w build`.
- Permissions — allow: pnpm, tsx, node, python, uv, gh, git, docker. Deny: `rm -rf /`, curl to non-allowlisted hosts.

## 8. Root Conveniences (`package.json` scripts)

`dev`, `build`, `lint`, `typecheck`, `test`, `compliance:scan`, `device:status`, `bi:forecast`, `format`. All wired to Turbo; non-implemented ones echo a clear "not yet implemented — see TODO.md".

## 9. CI Skeleton

- `.github/workflows/ci.yml` — matrix: {node-22, python-3.12}; runs lint, typecheck, test, build.
- `.github/workflows/security.yml` — gitleaks + trivy + pip-audit + npm audit; weekly schedule + on-push.

## 10. Explicit Non-Goals (this scaffold)

- No Appium flows.
- No translation/TTS pipeline.
- No Next.js pages beyond a placeholder.
- No NestJS modules beyond `AppModule` stub.
- No infra provisioning (Terraform is an empty placeholder).
- No database migrations.

## 11. First Commit

Single commit: `chore: scaffold monorepo, agents, harness, docs`.
Pushed to `main` (repo currently has only README + initial commit, no protected branch).

## 12. Open Questions / Follow-ups

- Postgres hosting: Supabase vs self-hosted on Hetzner — decide in sprint 1.
- Payment provider: Stripe (EU) confirmed; tax handling (Stripe Tax vs Merchant of Record) — defer.
- Which German MVNO for the SIM pool — defer to device-orchestrator's first ADR.

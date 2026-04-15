# CLAUDE.md — GlobalReel

> Project rules for Claude Code. Global rules in `~/.claude/rules/{common,web,typescript,python}/**` are inherited. This file overrides them where needed.

## Mission

GlobalReel is a B2B SaaS infrastructure for Chinese mid-tier short-drama teams publishing overseas. Defensibility rests on three moats — **every non-trivial decision must consider moat impact first, feature velocity second.**

### The three moats

1. **Physical network arbitrage** — real German consumer SIM cards + dynamic IP allocation. Datacenter / commercial-proxy IPs are the enemy.
2. **Automotive-grade quality engineering** — ASPICE V-model + an EU compliance engine. A single mass-ban kills trust; we ship fewer features, slower, but never lose accounts.
3. **Full-stack integration** — hardware swarm (Appium on ~10 real Android + SIMs) + AI pipeline (LLM translation, TTS, Veo 3.1 lip-sync) + cloud scheduler. The integration is the product.

## Repository Map

| Path | Owner | Language | Purpose |
|------|-------|----------|---------|
| `apps/web` | frontend | TS (Next.js 15) | Customer dashboard |
| `apps/api` | backend | TS (NestJS) | Cloud SaaS API |
| `apps/gateway` | backend | TS (Node) | Local Mac daemon — device orchestration bridge |
| `apps/automation` | automation | **Python** (Appium) | Device control, ADB, airplane-mode rotation |
| `apps/bi-worker` | data | **Python** (Polars/DuckDB) | Trend scraping + forecasting |
| `packages/shared-types` | shared | TS | Generated OpenAPI types |
| `packages/compliance` | shared | TS | EU compliance rule schemas |
| `packages/config` | shared | TS | ESLint / Prettier / tsconfig presets |
| `infra/` | devops | HCL/YAML | Docker Compose (dev) + Terraform (prod) |
| `docs/` | all | Markdown | PRD, architecture, specs, plans |
| `agents/` | meta | Markdown | Claude Code agent definitions |

## Language Split (hard rule)

- **TypeScript** is the default for any new code.
- **Python** is allowed *only* inside `apps/automation/` and `apps/bi-worker/`.
- Cross-boundary data travels through OpenAPI-generated types (`packages/shared-types` for TS, Pydantic models for Python). No ad-hoc JSON shapes across language boundaries.

## Agent Router

| Task intent | Invoke agent |
|-------------|--------------|
| Plan a sprint / feature / PRD breakdown | `planner` |
| Check script / video for EU legal risk | `compliance-scanner` |
| Anything inside `apps/automation/` or `apps/gateway/` device logic | `device-orchestrator` |
| Forecast CPM / CPS / funnel / trend scraping | `bi-forecaster` |
| Translate / dub / lip-sync / Veo | `ai-pipeline` |
| Review device / publisher config for detection risk | `antifingerprint-auditor` |
| Stripe / billing / JV rev-share accounting | `revenue-ops` |
| Write / translate technical-blog inbound content | `growth-writer` |
| Mass-ban incident / kill-switch / postmortem | `incident-responder` |

Before editing `apps/automation/` or `apps/gateway/` publishing code, invoke `device-orchestrator` **and** `antifingerprint-auditor`.

## Moat-Specific Rules (override global rules on conflict)

1. **No datacenter / commercial-proxy IPs, ever.** If you see a proxy allowlist, SOCKS config, residential-proxy API key, or VPN endpoint — stop and flag it. Publishing traffic only exits through the Mac Studio → Android + SIM path.
2. **Compliance pre-flight is non-negotiable.** Every code path that uploads, publishes, or schedules media must call `compliance-scanner` first. No feature flag can skip it in production.
3. **SIM rotation logging.** Log every airplane-mode toggle with timestamp + SHA-256(IMSI + per-device salt). Raw IMSI must never touch disk, logs, or telemetry.
4. **Fingerprint discipline.** Device fingerprints must come from real-device capture, never synthesized. New profiles need `antifingerprint-auditor` review.
5. **Kill-switch authority.** `apps/gateway` must expose a single-command kill-switch halting all device activity in <5 seconds. PRs touching this path require `incident-responder` sign-off.
6. **Human-in-the-loop translation.** AI-translated scripts must route through an editor-review queue before TTS. No full-auto publish on Tier 1.

## Commit & PR Conventions

- Conventional Commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`, `perf:`, `ci:`.
- No `Co-Authored-By` lines (matches global settings).
- PR title ≤ 70 chars. Body has Summary + Test Plan.
- Security-relevant PRs (`apps/gateway`, `apps/automation`, `packages/compliance`, auth/billing) require `security-reviewer` agent pass.

## Development Workflow

1. Research & reuse (`~/.claude/rules/common/development-workflow.md` §0).
2. Planner → TDD → Code review → Commit.
3. Spec-first for multi-file changes. Use `superpowers:brainstorming` → `superpowers:writing-plans`.

## Testing & Coverage

- Global minimum 80% coverage applies.
- `apps/automation` flows test against emulator + stub Appium server in CI; real-device tests run on the Mac Studio only.
- `packages/compliance` has a golden-rule corpus in `packages/compliance/fixtures/`.

## Non-Goals

- No user-facing AI-agent chat product. We're infrastructure.
- No C2C / creator-marketplace features.
- No crypto / tokenization.
- No Windows / Linux gateway support.

## Pointers

- Architecture: `docs/architecture.md`
- Moats: `docs/moats.md`
- GTM & pricing: `docs/go-to-market.md`
- Specs: `docs/superpowers/specs/`
- Plans: `docs/superpowers/plans/`

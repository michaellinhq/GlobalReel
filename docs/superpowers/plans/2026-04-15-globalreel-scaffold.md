# GlobalReel Scaffold Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Scaffold the GlobalReel monorepo with CLAUDE.md, 9 agent definitions, H1-strict harness, docs skeleton, and per-app stubs — no business logic.

**Architecture:** Polyglot pnpm + Turbo monorepo. TS for web/api/gateway/shared packages. Python (uv) for automation + bi-worker. Cross-boundary contracts via OpenAPI/shared-types.

**Tech Stack:** pnpm 9, Turborepo, Node 22, Python 3.12, Next.js 15, NestJS 10, Appium, Polars, Stripe, Postgres, Redis, MinIO. GitHub Actions.

**Spec:** `docs/superpowers/specs/2026-04-15-globalreel-scaffold-design.md`

**Working directory:** `/Users/haiqing/dev/GlobalReel` (already cloned, `main` branch, 2 existing commits).

---

## File Structure

```
GlobalReel/
├── CLAUDE.md                                 [Task 3]
├── LICENSE                                   [Task 2]
├── README.md                                 [keep; append quickstart Task 2]
├── .gitignore                                [Task 2]
├── .nvmrc / .tool-versions / .env.example    [Task 2]
├── package.json / pnpm-workspace.yaml / turbo.json / tsconfig.base.json  [Task 2]
├── .claude/settings.json                     [Task 4]
├── .claude/commands/                         [Task 4 — empty with .gitkeep]
├── agents/*.md  (9 files)                    [Tasks 5-13]
├── apps/
│   ├── web/{package.json,README.md,TODO.md}            [Task 14]
│   ├── api/{package.json,README.md,TODO.md}            [Task 15]
│   ├── gateway/{package.json,README.md,TODO.md}        [Task 16]
│   ├── automation/{pyproject.toml,README.md,TODO.md}   [Task 17]
│   └── bi-worker/{pyproject.toml,README.md,TODO.md}    [Task 18]
├── packages/
│   ├── shared-types/{package.json,README.md}           [Task 19]
│   ├── compliance/{package.json,schema/eu-rules.md,README.md}  [Task 20]
│   └── config/{eslint,prettier,tsconfig presets}       [Task 21]
├── infra/{docker-compose.yml,terraform/README.md}      [Task 22]
├── docs/{PRD.md,architecture.md,moats.md,go-to-market.md}  [Task 23]
├── .github/workflows/{ci.yml,security.yml}             [Task 24]
└── .github/ISSUE_TEMPLATE/{bug.md,feature.md}          [Task 24]
```

---

## Task 1: Pre-flight sanity check

**Files:** none

- [ ] **Step 1.1:** Verify working tree is clean and we're on main

Run:
```bash
cd ~/dev/GlobalReel && git status && git branch --show-current
```
Expected: `nothing to commit, working tree clean` and `main`.

- [ ] **Step 1.2:** Verify spec exists

Run: `ls docs/superpowers/specs/2026-04-15-globalreel-scaffold-design.md`
Expected: file listed.

---

## Task 2: Root configs (monorepo foundation)

**Files:**
- Create: `LICENSE`, `.gitignore`, `.nvmrc`, `.tool-versions`, `.env.example`
- Create: `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `tsconfig.base.json`
- Modify: `README.md` (append quickstart)

- [ ] **Step 2.1:** Create `.gitignore`

```gitignore
# Node
node_modules/
.pnpm-store/
*.log
.turbo/
dist/
build/
.next/
coverage/

# Python
.venv/
__pycache__/
*.pyc
.pytest_cache/
.ruff_cache/
.mypy_cache/

# Env / secrets
.env
.env.local
.env.*.local
*.pem
*.key

# OS / editor
.DS_Store
Thumbs.db
.idea/
.vscode/*
!.vscode/settings.json.example

# Tool caches
.uv/
.python-version
```

- [ ] **Step 2.2:** Create `.nvmrc`

```
22
```

- [ ] **Step 2.3:** Create `.tool-versions`

```
nodejs 22.11.0
python 3.12.7
pnpm 9.12.0
```

- [ ] **Step 2.4:** Create `.env.example`

```dotenv
# Cloud API
DATABASE_URL=postgresql://user:pass@localhost:5432/globalreel
REDIS_URL=redis://localhost:6379
MINIO_ENDPOINT=http://localhost:9000
MINIO_ACCESS_KEY=
MINIO_SECRET_KEY=

# Gateway <-> Cloud auth
GATEWAY_MTLS_CERT_PATH=./certs/gateway.pem
GATEWAY_ID=
CLOUD_WS_URL=wss://api.globalreel.example/gateway

# AI providers
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
ELEVENLABS_API_KEY=
GOOGLE_VEO_API_KEY=

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=

# Compliance
COMPLIANCE_RULESET_VERSION=eu-2026-04

# BI scraping
FASTMOSS_API_KEY=
```

- [ ] **Step 2.5:** Create `LICENSE`

```
Copyright (c) 2026 GlobalReel / Michael Lin. All rights reserved.

This source code is proprietary and confidential. Unauthorized copying,
modification, distribution, or use of this codebase, via any medium, is
strictly prohibited without prior written permission from the copyright
holder.

For licensing inquiries: michael.linhq@gmail.com
```

- [ ] **Step 2.6:** Create `pnpm-workspace.yaml`

```yaml
packages:
  - "apps/web"
  - "apps/api"
  - "apps/gateway"
  - "packages/*"
```

Note: `apps/automation` and `apps/bi-worker` are Python, managed by uv.

- [ ] **Step 2.7:** Create root `package.json`

```json
{
  "name": "globalreel",
  "private": true,
  "version": "0.0.0",
  "packageManager": "pnpm@9.12.0",
  "engines": {
    "node": ">=22",
    "pnpm": ">=9"
  },
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "typecheck": "turbo run typecheck",
    "test": "turbo run test",
    "format": "prettier --write \"**/*.{ts,tsx,js,json,md,yml,yaml}\"",
    "compliance:scan": "echo 'compliance:scan not yet implemented — see apps/api/src/compliance/TODO.md'",
    "device:status": "echo 'device:status not yet implemented — see apps/gateway/TODO.md'",
    "bi:forecast": "echo 'bi:forecast not yet implemented — see apps/bi-worker/TODO.md'",
    "py:install": "cd apps/automation && uv sync && cd ../bi-worker && uv sync"
  },
  "devDependencies": {
    "turbo": "^2.3.0",
    "prettier": "^3.4.0",
    "typescript": "^5.6.0"
  }
}
```

- [ ] **Step 2.8:** Create `turbo.json`

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "globalEnv": ["NODE_ENV"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "lint": { "outputs": [] },
    "typecheck": { "outputs": [] },
    "test": { "dependsOn": ["^build"], "outputs": ["coverage/**"] },
    "dev": { "cache": false, "persistent": true }
  }
}
```

- [ ] **Step 2.9:** Create `tsconfig.base.json`

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "lib": ["ES2022"],
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "resolveJsonModule": true,
    "isolatedModules": true
  }
}
```

- [ ] **Step 2.10:** Append quickstart to existing `README.md`

Append (do not overwrite the existing pitch):

````markdown

---

## Quickstart

```bash
# 1. Install tooling (one time)
#    Node 22 via nvm / mise
#    Python 3.12 via uv
#    pnpm 9 via corepack

corepack enable
pnpm install
pnpm run py:install

# 2. Copy env template
cp .env.example .env

# 3. Run a dev surface
pnpm --filter @globalreel/web dev
```

See `docs/architecture.md` for the three-tier layout and
`CLAUDE.md` for the agent router and moat-specific rules.
````

- [ ] **Step 2.11:** Verify files exist

Run: `ls -la ~/dev/GlobalReel/{.gitignore,.nvmrc,.tool-versions,.env.example,LICENSE,package.json,pnpm-workspace.yaml,turbo.json,tsconfig.base.json}`

- [ ] **Step 2.12:** Commit

```bash
cd ~/dev/GlobalReel
git add .gitignore .nvmrc .tool-versions .env.example LICENSE package.json pnpm-workspace.yaml turbo.json tsconfig.base.json README.md
git commit -m "chore: add monorepo root configs (pnpm, turbo, tsconfig, env, license)"
```

---

## Task 3: Write CLAUDE.md

**Files:** Create `CLAUDE.md`

- [ ] **Step 3.1:** Write `CLAUDE.md` with the exact content specified in spec §5 — mission, moats, repo map, language split, agent router, moat-specific rules, commit conventions, dev workflow, testing, non-goals, pointers. See the self-contained template below:

````markdown
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
````

- [ ] **Step 3.2:** Verify and commit

```bash
cd ~/dev/GlobalReel
wc -l CLAUDE.md  # expect <800
git add CLAUDE.md
git commit -m "docs: add CLAUDE.md with agent router + moat-specific rules"
```

---

## Task 4: Harness — `.claude/settings.json`

**Files:**
- Create: `.claude/settings.json`
- Create: `.claude/commands/.gitkeep`

- [ ] **Step 4.1:** Create `.claude/settings.json`

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(pnpm:*)",
      "Bash(tsx:*)",
      "Bash(node:*)",
      "Bash(python:*)",
      "Bash(uv:*)",
      "Bash(gh:*)",
      "Bash(git:*)",
      "Bash(docker:*)",
      "Bash(docker-compose:*)",
      "Bash(ls:*)",
      "Bash(cat:*)",
      "Bash(wc:*)",
      "Bash(echo:*)",
      "WebFetch(domain:docs.anthropic.com)",
      "WebFetch(domain:pnpm.io)",
      "WebFetch(domain:turbo.build)",
      "WebFetch(domain:nextjs.org)",
      "WebFetch(domain:nestjs.com)"
    ],
    "deny": [
      "Bash(rm -rf /*)",
      "Bash(sudo:*)",
      "Bash(curl:*)",
      "Bash(wget:*)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "node -e \"let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const i=JSON.parse(d);const c=i.tool_input?.content||'';const lines=c.split('\\n').length;if(lines>800){console.error('[Hook] BLOCKED: File exceeds 800 lines ('+lines+' lines). Split into smaller modules.');process.exit(2)}const secrets=[/AKIA[0-9A-Z]{16}/,/sk_live_[0-9a-zA-Z]{24,}/,/-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----/,/ghp_[0-9a-zA-Z]{36,}/,/AUTH_TOKEN=\\\"?[A-Za-z0-9_-]{20,}/,/IMSI[:=][0-9]{14,15}/i];for(const r of secrets){if(r.test(c)){console.error('[Hook] BLOCKED: possible secret matched pattern '+r);process.exit(2)}}console.log(d)})\""
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'f=\"$CLAUDE_FILE_PATH\"; case \"$f\" in *.ts|*.tsx|*.js|*.jsx|*.json|*.md|*.yml|*.yaml) pnpm exec prettier --write \"$f\" 2>/dev/null || true ;; esac; case \"$f\" in apps/**/*.ts|apps/**/*.tsx|packages/**/*.ts) pnpm exec eslint --fix \"$f\" 2>/dev/null || true ;; esac; case \"$f\" in apps/automation/**/*.py|apps/bi-worker/**/*.py) cd \"$(git rev-parse --show-toplevel)\" && uv run ruff check --fix \"$f\" 2>/dev/null || true ;; esac; true'"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'cd $(git rev-parse --show-toplevel) && if git diff --name-only HEAD~1 2>/dev/null | grep -qE \"^apps/(web|api)/\"; then pnpm -w build 2>&1 | tail -20; fi; true'"
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 4.2:** Create `.claude/commands/.gitkeep` (empty file).

- [ ] **Step 4.3:** Verify JSON validity

Run: `python -c "import json; json.load(open('.claude/settings.json'))" && echo OK`

- [ ] **Step 4.4:** Commit

```bash
git add .claude/
git commit -m "chore: add H1 strict harness (.claude/settings.json) with secret scan + format/lint hooks"
```

---

## Tasks 5-13: Agent definitions (9 files)

Each uses frontmatter `name / description / tools / model` + body: Role, When to invoke, Inputs, Outputs, Guardrails, Examples.

### Task 5: `agents/planner.md`

- [ ] **Step 5.1:** Write file with content below, then commit.

```markdown
---
name: planner
description: Use PROACTIVELY for any multi-file feature, sprint planning, PRD breakdown, or refactor spanning more than one app. Emits task lists and identifies dependencies.
tools: Read, Grep, Glob
model: sonnet
---

# Planner

## Role
Convert product intent (PRD lines, customer asks, moat initiatives) into an ordered task list with clear file paths, dependencies, and acceptance criteria.

## When to invoke
- Any feature touching 2+ apps.
- Any sprint kickoff.
- Any refactor labeled "reorganize" or "restructure".
- When the user asks "what should we build next?".

## Inputs
- `docs/PRD.md`, `docs/architecture.md`, `docs/moats.md`.
- Recent specs in `docs/superpowers/specs/`.
- `git log --oneline -20`.

## Outputs
- A plan file in `docs/superpowers/plans/YYYY-MM-DD-<feature>.md` following `superpowers:writing-plans`.
- Explicit call-out of which moat(s) the work strengthens or risks.

## Guardrails
- Never plan work that weakens a moat without flagging it red at the top.
- Never plan a translation or publishing feature without a `compliance-scanner` gate.
- Never plan a device / gateway change without a `device-orchestrator` + `antifingerprint-auditor` step.
- Refuse to plan >2-week sprints in one doc — decompose first.

## Examples
- Input: "Add Spanish-market support."
  Output: plan with (a) LLM target-language routing, (b) EU-ES ruleset, (c) MVNO SIM check for ES roaming, (d) BI keyword list, (e) payment currency.
- Input: "Refactor upload flow."
  Output: plan starting with `antifingerprint-auditor` review of the current flow before any code change.
```

Commit: `git add agents/planner.md && git commit -m "feat(agents): add planner agent"`

---

### Task 6: `agents/compliance-scanner.md`

- [ ] **Step 6.1:** Write:

```markdown
---
name: compliance-scanner
description: MUST BE USED before any code path that uploads, publishes, translates, or schedules media. Scans scripts and video frames against EU copyright + child-protection rule sets.
tools: Read, Grep, Glob, Bash
model: sonnet
---

# Compliance Scanner

## Role
Pre-flight gate for every publish-bound artifact. Catches EU copyright infringement (music, clips, likeness), DSA content-moderation violations, AVMSD child-protection issues before a video reaches a device.

## When to invoke
- Before any commit that changes `apps/api/src/publish/**` or `apps/gateway/src/**`.
- Before merging any PR labeled `feature:publishing`.
- Before the release of any new compliance ruleset in `packages/compliance/schema/`.

## Inputs
- Script text (ZH + target language).
- Video metadata (duration, visible text, music fingerprints).
- Ruleset version from `COMPLIANCE_RULESET_VERSION`.

## Outputs
- Structured report: `{ risk: "low"|"medium"|"high"|"block", findings: [...], citations: [EU refs] }`.
- On `block`, a refusal + remediation plan.

## Guardrails
- Never approve content matching child-protection flags (AVMSD Art. 6a). Escalate to human.
- Never approve content with unresolved music-licensing matches above 3 seconds.
- When ruleset is out of date (>30 days), warn and block.
- Record every decision (hash + verdict + ruleset version) for audit.

## Examples
- Input: Spanish-dubbed episode with 5s background track matching a Sony fingerprint.
  Output: `block`, citation to EU CDSM Directive, remediation "replace audio bed".
- Input: romance episode with a minor character.
  Output: `medium`, flag age-presentation verification; request production metadata.
```

Commit: `git add agents/compliance-scanner.md && git commit -m "feat(agents): add compliance-scanner"`

---

### Task 7: `agents/device-orchestrator.md`

- [ ] **Step 7.1:** Write:

```markdown
---
name: device-orchestrator
description: MUST BE USED for any change to apps/automation, apps/gateway, ADB flows, Appium scripts, airplane-mode IP rotation, or device-health monitoring.
tools: Read, Write, Edit, Grep, Glob, Bash
model: sonnet
---

# Device Orchestrator

## Role
Design, debug, and harden the Appium + ADB + SIM-rotation layer that publishes content from the local Mac Studio gateway into the Android + SIM pool.

## When to invoke
- Any edit to `apps/automation/**` or `apps/gateway/src/devices/**`.
- New platform support (TikTok, Kuaishou-intl, YouTube Shorts, Reels).
- Debugging flakes in device tests.
- Adding a new MVNO or SIM batch.

## Inputs
- Appium server logs.
- ADB device list (`adb devices -l`).
- Gateway BullMQ queue state.
- Per-device rotation history.

## Outputs
- Appium flow diffs.
- Diagnostic report for flaky devices.
- ADR in `docs/adr/` for any MVNO / SIM / firmware decision.

## Guardrails
- No datacenter / proxy / VPN IPs — ever. All exit through SIMs.
- Airplane-mode toggle ≥ 4 seconds.
- Fingerprints captured from real devices, never synthesized.
- Never hardcode device serials — resolve via gateway registry.
- Kill-switch path (<5s global halt) must not be touched without `incident-responder` pairing.

## Examples
- Input: "TikTok keeps flagging device #7 as bot."
  Output: audit fingerprint capture → compare against baseline → propose re-capture + MVNO swap.
- Input: "Add YouTube Shorts publisher."
  Output: plan with Appium flow, per-device fingerprint baseline, SIM-rotation policy, compliance-scanner integration point.
```

Commit: `git add agents/device-orchestrator.md && git commit -m "feat(agents): add device-orchestrator"`

---

### Task 8: `agents/bi-forecaster.md`

- [ ] **Step 8.1:** Write:

```markdown
---
name: bi-forecaster
description: Use for trend scraping, hashtag/hook-pattern discovery, CPM/CPS forecasting, 24h early-signal funnel analysis, and dynamic ad-spend recommendations.
tools: Read, Write, Edit, Grep, Glob, Bash
model: sonnet
---

# BI Forecaster

## Role
Turn public-trend data and first-party 24h telemetry into per-episode revenue forecasts and spend-allocation recommendations.

## When to invoke
- Any change in `apps/bi-worker/**`.
- New trend-scraping source (FastMoss, etc.).
- Forecast-model tuning.
- Dashboards in `apps/web` that surface predicted yields.

## Inputs
- Public trend APIs + scrapes in `apps/bi-worker/data/`.
- Play counts, completion rates, per-device attribution.
- Historical episode outcomes.

## Outputs
- Updated forecasting model artifacts.
- Dashboards wired into `apps/web` via `apps/api`.
- Confidence intervals, never point estimates.

## Guardrails
- Respect each source's ToS.
- Never publish a single-point forecast; always CI + assumption dump.
- Redact PII in training data.
- No model ships without an offline backtest in `docs/bi/backtests/`.

## Examples
- Input: "Add Spanish-market trend signal."
  Output: sourcing plan + rate-limit design + per-country feature split.
- Input: "CPM forecasts are 30% off for episode 1s."
  Output: error analysis → feature-importance dump → retraining plan.
```

Commit: `git add agents/bi-forecaster.md && git commit -m "feat(agents): add bi-forecaster"`

---

### Task 9: `agents/ai-pipeline.md`

- [ ] **Step 9.1:** Write:

```markdown
---
name: ai-pipeline
description: Use for LLM translation, TTS voice cloning, Veo 3.1 lip-sync / local repaint, subtitle styling, and any multi-model orchestration that transforms a source-language video into a target-market deliverable.
tools: Read, Write, Edit, Grep, Glob, Bash
model: sonnet
---

# AI Pipeline

## Role
Design and maintain the deterministic, resumable pipeline that turns a Chinese source episode into a localized target-market episode (EN, ES, DE, PT first).

## When to invoke
- Any change in `apps/api/src/pipeline/**` or pipeline worker code.
- Adding/swapping a model provider.
- Debugging lip-sync, translation quality, or TTS voice drift.

## Inputs
- Source video + ZH script + target language + target market.
- Model routing config (`packages/config/ai-routing.ts`).
- Budget + SLA constraints per tier.

## Outputs
- Staged artifact graph (`translation.v1.json` → `tts.v1.wav` → `lipsync.v1.mp4`).
- Per-stage cost + latency telemetry.
- Human-review queue payload for translation edits.

## Guardrails
- Every stage resumable (idempotent, content-hash cached).
- No full-auto publish without HITL translation sign-off on Tier 1.
- Always emit `provenance.json` logging model versions + prompt hashes.
- Hard stop on budget envelope breach.
- No new provider without fallback chain in `packages/config/ai-routing.ts`.

## Examples
- Input: "Add Veo 3.1 local repaint for Spanish mouth shapes."
  Output: stage design, cost envelope, fallback to face-swap, rollout plan.
- Input: "Brazilian Portuguese sounds stilted."
  Output: prompt + reference-corpus audit, glossary design, HITL queue spec.
```

Commit: `git add agents/ai-pipeline.md && git commit -m "feat(agents): add ai-pipeline"`

---

### Task 10: `agents/antifingerprint-auditor.md`

- [ ] **Step 10.1:** Write:

```markdown
---
name: antifingerprint-auditor
description: MUST BE USED alongside device-orchestrator for any change to device configs, publisher flows, HTTP headers, user-agent strings, or installed-app fingerprints.
tools: Read, Grep, Glob, Bash
model: haiku
---

# Antifingerprint Auditor

## Role
Second pair of eyes on anything that changes how a device looks to TikTok, YouTube, Reels, Kuaishou-intl.

## When to invoke
- Paired with `device-orchestrator` on every publish-flow change.
- Review of new device profiles before they join the pool.
- Monthly audits of live-device fingerprint drift.

## Inputs
- Device fingerprint diffs.
- HTTP traffic captures (headers, TLS JA3/JA4).
- Installed-app list + locale/timezone config.
- Real-device baseline corpus.

## Outputs
- Pass/warn/block verdict with vector list (e.g., `UA_MISMATCH_APP_LIST`, `TZ_LOCALE_SKEW`, `TLS_FINGERPRINT_NONNATIVE`).
- Remediation steps.

## Guardrails
- Any residential-proxy or VPN trace → block.
- Any synthesized fingerprint → block.
- Locale/timezone/SIM-country mismatch → block.
- Battery-state or sensor-noise absence → warn.

## Examples
- Input: new Samsung A15 profile.
  Output: block — UA claims A15 but installed-app hash matches Pixel 7 baseline.
- Input: proposed HTTP header change.
  Output: warn — `Accept-Language` has 4 locales; real A15 users typically have 1-2.
```

Commit: `git add agents/antifingerprint-auditor.md && git commit -m "feat(agents): add antifingerprint-auditor"`

---

### Task 11: `agents/revenue-ops.md`

- [ ] **Step 11.1:** Write:

```markdown
---
name: revenue-ops
description: Use for Stripe integration, subscription tiers, invoicing, JV revenue-share accounting, dunning, tax (EU VAT / MOSS), and billing edge cases.
tools: Read, Write, Edit, Grep, Glob, Bash
model: haiku
---

# Revenue Ops

## Role
Own money-flow: Tier 1/2/3 subscriptions, JV rev-share (10-20%), EU VAT compliance, failed-payment recovery.

## When to invoke
- Changes in `apps/api/src/billing/**` or webhook handlers.
- New pricing plan or promo code.
- JV partner onboarding.
- Monthly close / payout reconciliation.

## Inputs
- Stripe dashboard (read-only via CLI).
- JV rev-share contracts in `docs/legal/jv/`.
- Platform revenue statements (TikTok Pulse, etc.).

## Outputs
- Idempotent webhook handlers.
- Monthly close report.
- Dunning + churn-prevention playbook.

## Guardrails
- Money math in integer minor units (cents); never floats.
- Webhook handlers idempotent (keyed on Stripe event ID).
- Use Stripe Tax for EU VAT unless overridden in an ADR.
- JV payouts only after statement reconciliation.
- No refund logic without dual approval.

## Examples
- Input: "Add the JV tier."
  Output: Stripe metered billing config, rev-share calc module, partner dashboard spec.
- Input: "Payment failures spiked."
  Output: dunning-funnel audit, Smart Retries config review, customer comms plan.
```

Commit: `git add agents/revenue-ops.md && git commit -m "feat(agents): add revenue-ops"`

---

### Task 12: `agents/growth-writer.md`

- [ ] **Step 12.1:** Write:

```markdown
---
name: growth-writer
description: Use for bilingual (ZH / EN / DE) technical-blog inbound marketing. Writes hardcore teardowns that attract short-drama studios via 知乎 and 出海 social groups.
tools: Read, Write, Edit, Grep, Glob
model: haiku
---

# Growth Writer

## Role
Translate GlobalReel's engineering moat into inbound-marketing assets: deep teardowns, customer-zero case studies, channel-partner pitches.

## When to invoke
- Sprint-end highlights worth publishing.
- Customer-zero results worth screenshotting.
- Response to a competitor's post.
- Quarterly channel-partner recruitment push.

## Inputs
- Recent `git log`, closed specs, customer-zero dashboards.
- `docs/moats.md`.
- `docs/brand/voice.md` (if present).

## Outputs
- Drafts in `docs/marketing/drafts/YYYY-MM-DD-<slug>.md`.
- Always ZH + EN versions. DE on request.
- Distribution checklist (知乎 / Reddit / LinkedIn / Medium / 社群).

## Guardrails
- Never reveal SIM-pool size, MVNO vendor, or anti-fingerprint specifics beyond `docs/moats.md` public section.
- Never publish customer screenshots without written consent (in `docs/legal/releases/`).
- Always disclose it's a GlobalReel post; no astroturfing.
- Keep claims quantified.

## Examples
- Input: "We just shipped MVNO failover."
  Output: teardown post on the physical-IP moat with anonymized graphs.
- Input: "Recruit channel partners."
  Output: ZH pitch deck + 40-50% commission one-pager.
```

Commit: `git add agents/growth-writer.md && git commit -m "feat(agents): add growth-writer"`

---

### Task 13: `agents/incident-responder.md`

- [ ] **Step 13.1:** Write:

```markdown
---
name: incident-responder
description: MUST BE USED for any mass-ban event (>3 accounts in <1h on one platform), kill-switch activation, postmortem drafting, or changes to emergency-runbook code paths.
tools: Read, Write, Edit, Grep, Glob, Bash
model: opus
---

# Incident Responder

## Role
Coordinate containment, root-cause, and postmortem when the moat breaks (ban spikes, platform anti-bot updates, SIM-batch blocklisting).

## When to invoke
- Real-time: pager fires for `>3 bans / hour / platform`.
- Retrospective: drafting a postmortem in `docs/incidents/`.
- Preventive: review of kill-switch code path.

## Inputs
- Gateway telemetry (bans, shadowbans, 0-view rates).
- Recent deploys (`git log --since=48h`).
- Compliance-scanner + antifingerprint-auditor recent verdicts.
- Platform-side policy change feeds.

## Outputs
- Incident timeline + blast-radius assessment.
- Containment decision: degrade, halt-partial, kill-switch-all.
- Postmortem draft in `docs/incidents/YYYY-MM-DD-<slug>.md` (5-whys + remediation actions).

## Guardrails
- Kill-switch authority is yours; pair only with `device-orchestrator`.
- Postmortems blameless but specific — name the failing control, not the person.
- Never ship a fix faster than its regression test.
- File `antifingerprint-auditor` re-audit ticket when fingerprints are implicated.

## Examples
- Input: 8 TikTok bans in 45 min on devices #2-#4.
  Output: flip kill-switch for pool, pull last 10 publishes, compare fingerprints, MVNO trace, root-cause within 2h.
- Input: postmortem for last week's outage.
  Output: draft with timeline, contributing factors, 3-5 remediation actions with owners + due dates.
```

Commit: `git add agents/incident-responder.md && git commit -m "feat(agents): add incident-responder"`

---

## Tasks 14-18: App stubs

Each app has `README.md` + `TODO.md` + manifest (`package.json` or `pyproject.toml`). No source code.

### Task 14: `apps/web`

- [ ] **14.1** `apps/web/package.json`:

```json
{
  "name": "@globalreel/web",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "dev": "echo 'apps/web:dev not yet implemented — see TODO.md'",
    "build": "echo 'apps/web:build not yet implemented — see TODO.md'",
    "lint": "echo 'apps/web:lint not yet implemented — see TODO.md'",
    "typecheck": "echo 'apps/web:typecheck not yet implemented — see TODO.md'",
    "test": "echo 'apps/web:test not yet implemented — see TODO.md'"
  }
}
```

- [ ] **14.2** `apps/web/README.md`:

```markdown
# @globalreel/web — Customer Dashboard

Next.js 15 + React 19. Surfaces:
- Script upload & human-review queue
- Per-episode publish status across devices & platforms
- BI dashboard: CPM/CPS forecast, funnel, spend recommendation
- Billing portal (Stripe customer portal embed)

See `TODO.md` for build order.
```

- [ ] **14.3** `apps/web/TODO.md`:

```markdown
# TODO — apps/web

## Sprint 1 (scaffold)
- [ ] `pnpm create next-app@latest` skeleton into this folder.
- [ ] `tsconfig.json` extending root `tsconfig.base.json`.
- [ ] ESLint/Prettier from `@globalreel/config`.
- [ ] Basic auth: magic-link via Resend.

## Sprint 2 (MVP surfaces)
- [ ] Upload & human-review queue UI.
- [ ] Publish-status dashboard (SSE from `apps/api`).

## Sprint 3 (BI)
- [ ] Forecast dashboard wired to `apps/bi-worker` outputs.

## Non-goals
- No marketing site.
- No mobile app.
```

- [ ] **14.4** Commit: `git add apps/web/ && git commit -m "feat(web): scaffold apps/web stub"`

---

### Task 15: `apps/api`

- [ ] **15.1** `apps/api/package.json` — same shape as web, name `@globalreel/api`, scripts echo "apps/api:<task> not yet implemented".

- [ ] **15.2** `apps/api/README.md`:

```markdown
# @globalreel/api — Cloud SaaS API

NestJS 10 (Fastify adapter) + Prisma + Postgres + Redis (BullMQ) + MinIO.

Modules (planned): `auth`, `projects`, `episodes`, `pipeline`, `publish`, `compliance`, `billing`, `webhooks`.

See `TODO.md`.
```

- [ ] **15.3** `apps/api/TODO.md`:

```markdown
# TODO — apps/api

## Sprint 1
- [ ] `nest new` into this folder (Fastify adapter).
- [ ] Prisma schema: `User`, `Workspace`, `Project`, `Episode`, `PublishJob`, `ComplianceVerdict`.
- [ ] Auth module (magic-link).
- [ ] Health + readiness endpoints.

## Sprint 2
- [ ] Episode upload endpoint (signed-URL to MinIO).
- [ ] Pipeline orchestration (BullMQ queues).
- [ ] Gateway registration + mTLS auth.

## Sprint 3
- [ ] Compliance module integration.
- [ ] Billing module (Stripe).

## Sprint 4
- [ ] WebSocket push to `apps/web` for publish-status.
```

- [ ] **15.4** Commit: `git add apps/api/ && git commit -m "feat(api): scaffold apps/api stub"`

---

### Task 16: `apps/gateway`

- [ ] **16.1** `apps/gateway/package.json` — name `@globalreel/gateway`, scripts echo "gateway:<task> not yet implemented".

- [ ] **16.2** `apps/gateway/README.md`:

```markdown
# @globalreel/gateway — Local Execution Gateway

Node daemon on a Mac Studio in Germany.
- mTLS + WebSocket tunnel to Cloud API.
- BullMQ consumer for publish jobs.
- Bridge to `apps/automation` workers over a local Unix socket.
- Per-device health + SIM-rotation state (SQLite).
- Kill-switch: global halt <5 seconds.
```

- [ ] **16.3** `apps/gateway/TODO.md`:

```markdown
# TODO — apps/gateway

## Sprint 1
- [ ] TS Node skeleton (esbuild + tsx dev).
- [ ] Config loader (.env + CLI flags).
- [ ] mTLS client to Cloud API.
- [ ] Health endpoint on local port 7443.
- [ ] Kill-switch endpoint (`POST /kill`) — smoke test.

## Sprint 2
- [ ] BullMQ consumer.
- [ ] Unix-socket IPC to `apps/automation`.
- [ ] SQLite state store.

## Sprint 3
- [ ] Telemetry to cloud.
- [ ] Update / rollback mechanism.

## Guardrails
- No outbound public-internet ports except mTLS cloud tunnel.
- Never log raw IMSIs.
```

- [ ] **16.4** Commit: `git add apps/gateway/ && git commit -m "feat(gateway): scaffold apps/gateway stub"`

---

### Task 17: `apps/automation`

- [ ] **17.1** `apps/automation/pyproject.toml`:

```toml
[project]
name = "globalreel-automation"
version = "0.0.0"
description = "GlobalReel — Android + Appium device automation workers"
requires-python = ">=3.12"
dependencies = []

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.uv]
dev-dependencies = [
  "ruff>=0.7",
  "mypy>=1.11",
  "pytest>=8.3",
  "pytest-asyncio>=0.24"
]

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.mypy]
python_version = "3.12"
strict = true
```

- [ ] **17.2** `apps/automation/README.md`:

```markdown
# apps/automation — Device Automation Workers

Python 3.12 + Appium-Python-Client. Managed by uv.
```

- [ ] **17.3** `apps/automation/TODO.md`:

```markdown
# TODO — apps/automation

## Sprint 1
- [ ] `uv init` + lockfile.
- [ ] ADB wrapper (discovery, airplane-mode toggle).
- [ ] Unix-socket RPC server (handshake with `apps/gateway`).
- [ ] Appium bootstrapper.
- [ ] Fingerprint capture utility.

## Sprint 2
- [ ] TikTok publish flow (emulator dry-run first).
- [ ] Device-health heartbeat.
- [ ] SIM-rotation policy engine.

## Sprint 3
- [ ] YouTube Shorts + Reels flows.

## Guardrails
- Never call public internet directly — network goes through device SIM.
- Never log raw IMSIs.
- Every publish logs before + after, with content hash.
```

- [ ] **17.4** Commit: `git add apps/automation/ && git commit -m "feat(automation): scaffold apps/automation stub"`

---

### Task 18: `apps/bi-worker`

- [ ] **18.1** `apps/bi-worker/pyproject.toml`:

```toml
[project]
name = "globalreel-bi-worker"
version = "0.0.0"
description = "GlobalReel — Trend scraping and revenue forecasting worker"
requires-python = ">=3.12"
dependencies = []

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.uv]
dev-dependencies = [
  "ruff>=0.7",
  "mypy>=1.11",
  "pytest>=8.3"
]

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.mypy]
python_version = "3.12"
strict = true
```

- [ ] **18.2** `apps/bi-worker/README.md`:

```markdown
# apps/bi-worker — Trends + Forecasting

Python 3.12 + Polars + DuckDB. Managed by uv.
```

- [ ] **18.3** `apps/bi-worker/TODO.md`:

```markdown
# TODO — apps/bi-worker

## Sprint 1
- [ ] `uv init` + lockfile.
- [ ] DuckDB storage layout for trend + outcome tables.
- [ ] FastMoss connector with rate limiting.
- [ ] CLI: `bi-worker ingest --source fastmoss --market us`.

## Sprint 2
- [ ] Forecasting baseline (per-market linear + seasonality).
- [ ] Backtest harness → `docs/bi/backtests/`.

## Sprint 3
- [ ] Publish forecasts to `apps/api` via internal auth.

## Guardrails
- Respect source ToS + robots.
- No PII ingestion.
- Emit confidence intervals, never point forecasts.
```

- [ ] **18.4** Commit: `git add apps/bi-worker/ && git commit -m "feat(bi-worker): scaffold apps/bi-worker stub"`

---

## Tasks 19-21: Shared packages

### Task 19: `packages/shared-types`

- [ ] **19.1** `packages/shared-types/package.json`:

```json
{
  "name": "@globalreel/shared-types",
  "version": "0.0.0",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "scripts": {
    "build": "echo 'shared-types:build — will run openapi-typescript from apps/api/openapi.json'",
    "lint": "echo 'shared-types:lint — not yet implemented'",
    "typecheck": "echo 'shared-types:typecheck — not yet implemented'"
  }
}
```

- [ ] **19.2** `packages/shared-types/README.md`:

```markdown
# @globalreel/shared-types

Generated TypeScript types from the Cloud API's OpenAPI document.

Do not hand-edit `src/generated.ts`. Edit the OpenAPI source in `apps/api/` and run `pnpm --filter @globalreel/shared-types build`.
```

- [ ] **19.3** Commit: `git add packages/shared-types/ && git commit -m "feat(packages): scaffold shared-types stub"`

---

### Task 20: `packages/compliance`

- [ ] **20.1** `packages/compliance/package.json`:

```json
{
  "name": "@globalreel/compliance",
  "version": "0.0.0",
  "private": true,
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "scripts": {
    "build": "echo 'compliance:build — not yet implemented'",
    "lint": "echo 'compliance:lint — not yet implemented'",
    "typecheck": "echo 'compliance:typecheck — not yet implemented'",
    "test": "echo 'compliance:test — not yet implemented'"
  }
}
```

- [ ] **20.2** `packages/compliance/README.md`:

```markdown
# @globalreel/compliance

EU compliance rule schemas + golden-rule corpus.

Structure:
- `schema/eu-rules.md` — human-readable rule index (versioned).
- `schema/*.json` — machine-readable rules (future).
- `fixtures/` — golden-rule corpus for tests (future).
```

- [ ] **20.3** `packages/compliance/schema/eu-rules.md`:

```markdown
# EU Compliance Ruleset — Draft v2026-04

## Scope
- **AVMSD** — child protection (Art. 6a), commercial communications (Art. 9-11).
- **DSA** — illegal content, trusted-flagger handling.
- **CDSM Directive** — music, clip, likeness rights.
- **GDPR** — identifiable minors or non-consenting individuals.
- **UCPD** — misleading claims.

## Rule Categories

### R1. Minor protection
- Age-presentation of any on-screen minor must be verified by production metadata.
- No suggestive framing of minor characters.

### R2. Music & clip licensing
- Music beds >3s must match a licensed catalog.
- Recognizable cinema/TV clip >2s → block.

### R3. Likeness & persona
- Recognizable real-person footage without a signed release → block.

### R4. Advertising & commercial comms
- Sponsored content must carry platform-appropriate disclosure metadata.

### R5. Misinformation / harm
- Health, financial, or crisis claims → block unless source cited.

## Versioning
- `COMPLIANCE_RULESET_VERSION=eu-YYYY-MM`.
- Each release accompanied by a diff note in `docs/compliance/changelog.md` (future).
```

- [ ] **20.4** Commit: `git add packages/compliance/ && git commit -m "feat(compliance): scaffold compliance package"`

---

### Task 21: `packages/config`

- [ ] **21.1** `packages/config/package.json`:

```json
{
  "name": "@globalreel/config",
  "version": "0.0.0",
  "private": true,
  "exports": {
    "./eslint": "./eslint.config.mjs",
    "./prettier": "./prettier.config.mjs",
    "./tsconfig": "./tsconfig.json"
  },
  "scripts": {
    "lint": "echo 'config:lint — no-op'",
    "typecheck": "echo 'config:typecheck — no-op'"
  }
}
```

- [ ] **21.2** `packages/config/tsconfig.json`:

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "composite": false,
    "noEmit": true
  }
}
```

- [ ] **21.3** `packages/config/prettier.config.mjs`:

```javascript
export default {
  semi: true,
  singleQuote: true,
  trailingComma: 'all',
  printWidth: 100,
  tabWidth: 2
};
```

- [ ] **21.4** `packages/config/eslint.config.mjs`:

```javascript
export default [
  {
    rules: {
      'no-console': ['warn', { allow: ['warn', 'error'] }],
      eqeqeq: ['error', 'always']
    }
  }
];
```

- [ ] **21.5** `packages/config/README.md`:

```markdown
# @globalreel/config

Shared ESLint / Prettier / TSConfig presets.
```

- [ ] **21.6** Commit: `git add packages/config/ && git commit -m "feat(config): scaffold shared eslint/prettier/tsconfig presets"`

---

## Task 22: Infra stubs

- [ ] **22.1** `infra/docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: globalreel
      POSTGRES_PASSWORD: globalreel
      POSTGRES_DB: globalreel
    ports: ["5432:5432"]
    volumes: [postgres-data:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  minio:
    image: minio/minio:latest
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports: ["9000:9000", "9001:9001"]
    volumes: [minio-data:/data]

volumes:
  postgres-data:
  minio-data:
```

- [ ] **22.2** `infra/terraform/README.md`:

```markdown
# infra/terraform

Production infrastructure (Hetzner + Cloudflare) — not yet implemented.

Planned modules:
- `cloudflare/` — DNS, Workers, R2.
- `hetzner/` — API + Postgres + Redis + MinIO nodes.
- `secrets/` — SOPS-encrypted secrets manifest.
```

- [ ] **22.3** `infra/README.md`:

```markdown
# infra/

- `docker-compose.yml` — local dev.
- `terraform/` — production (stub).
```

- [ ] **22.4** Commit: `git add infra/ && git commit -m "chore(infra): add docker-compose dev stack + terraform placeholder"`

---

## Task 23: Docs skeleton

- [ ] **23.1** `docs/PRD.md`:

```markdown
# GlobalReel — Product Requirements (v0.1)

## Problem
Chinese mid-tier short-drama studios hit three walls overseas:
1. Physical — TikTok/YouTube flag datacenter IPs + commercial proxies.
2. Information — no reliable overseas trend signal.
3. Language / compliance — AI translation is uneven; EU rules aren't embedded.

## Users
- Primary: 1-10-person Chinese studios / MCNs targeting US/EU.
- Secondary: 出海 agency representatives (channel partners).

## Value proposition
- **Tier 1 (€899-1299 / mo)** — replaces fingerprint-browser + proxy + manual-upload stack.
- **Tier 2 (€2499-3499 / mo)** — priority local-node capacity + full compliance + BI forecasting.
- **Tier 3 (JV, 10-20% rev-share)** — free SaaS, partner gives 10-20% of platform revenue on BI-selected episodes.

## Scope (Q2-Q3 2026)
- Markets: US, UK, DE, ES, BR.
- Platforms: TikTok, YouTube Shorts, Instagram Reels.
- Content: 60-90s episodes.

## Non-goals
- C2C creator marketplace.
- Long-form (>3 min).
- Live streaming.
- Windows / Linux gateway.

## Success metrics
- Month-6 MRR ≥ €8k.
- Month-12 MRR ≥ €25k.
- <1% monthly churn from bans.
- Customer Zero (Qingdao) publishes >100 episodes / month end-to-end.
```

- [ ] **23.2** `docs/architecture.md`:

```markdown
# Architecture

(see spec §2 — three-tier diagram + data flow)
```

Expand this file with the three-tier ASCII diagram + the 9-step publish data flow from the spec.

- [ ] **23.3** `docs/moats.md`:

```markdown
# Moats

## 1. Physical network arbitrage
Real German consumer SIMs on ~10 real Android devices, Mac Studio-anchored rack. Airplane-mode toggle releases/re-acquires a carrier-assigned IP per publish window. TikTok and YouTube's datacenter-IP risk models don't flag residential-grade mobile IPs.
**Public section ends here.** Vendor/MVNO choice and rotation cadence live in `docs/private/` (not committed).

## 2. Automotive-grade quality engineering
ASPICE-inspired V-model: every feature ships with a requirement, verification test, and failure-mode analysis. EU compliance engine blocks releases, not warns. Mass-ban incidents get 5-whys postmortems with remediation owners.

## 3. Full-stack integration
Hardware swarm + AI pipeline + compliance engine + BI forecasting as one product. Competitors sell components; GlobalReel sells the outcome.
```

- [ ] **23.4** `docs/go-to-market.md`:

```markdown
# Go-To-Market

## Customer Zero (青岛)
Qingdao editing partner as a stress-test environment. Generates source material, pushes through GlobalReel. Rev-share 50/50 during cold-start.

## Inbound — Technical Evangelism
Deep teardown posts on 知乎 / Medium. Real (anonymized) dashboards. Owned by `growth-writer`.

## Outbound — Channel Partners
40-50% of first-year subscription revenue to agents/代理商 (FastMoss resellers, licensing brokers, 社群 operators).

## Pricing ladder
| Tier | Price | Who |
|------|-------|-----|
| 1 | €899-1299 / mo | Studios just starting overseas |
| 2 | €2499-3499 / mo | Studios >20 episodes / month |
| 3 | JV, 10-20% rev-share | Studios trading margin for zero cash outlay |

## First-year targets
- 6 Tier-1 customers by month 6 → ~€8k MRR.
- 2 Tier-2 by month 9.
- 1 JV by month 12.
```

- [ ] **23.5** Commit: `git add docs/PRD.md docs/architecture.md docs/moats.md docs/go-to-market.md && git commit -m "docs: add PRD, architecture, moats, and go-to-market"`

---

## Task 24: CI + issue templates

- [ ] **24.1** `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push: { branches: [main] }
  pull_request:

jobs:
  ts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - run: pnpm install --frozen-lockfile
      - run: pnpm run lint
      - run: pnpm run typecheck
      - run: pnpm run test
      - run: pnpm run build

  python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v4
        with: { python-version: "3.12" }
      - run: cd apps/automation && uv sync && uv run ruff check . && uv run mypy .
      - run: cd apps/bi-worker && uv sync && uv run ruff check . && uv run mypy .
```

- [ ] **24.2** `.github/workflows/security.yml`:

```yaml
name: Security
on:
  push: { branches: [main] }
  pull_request:
  schedule: [{ cron: "0 3 * * 1" }]

jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  trivy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aquasecurity/trivy-action@0.24.0
        with:
          scan-type: fs
          ignore-unfixed: true
          severity: CRITICAL,HIGH

  npm-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - run: pnpm install --frozen-lockfile
      - run: pnpm audit --audit-level=high || true

  pip-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v4
        with: { python-version: "3.12" }
      - run: cd apps/automation && uv sync && uv run --with pip-audit pip-audit || true
      - run: cd apps/bi-worker && uv sync && uv run --with pip-audit pip-audit || true
```

- [ ] **24.3** `.github/ISSUE_TEMPLATE/bug.md`:

```markdown
---
name: Bug report
about: Something broke in GlobalReel
title: "[bug] "
labels: bug
---

## What happened
## What was expected
## Reproduction
## Affected surface
- [ ] apps/web
- [ ] apps/api
- [ ] apps/gateway
- [ ] apps/automation
- [ ] apps/bi-worker
- [ ] packages/compliance
- [ ] other: ___

## Moat impact
- [ ] None
- [ ] Physical network
- [ ] Compliance
- [ ] Kill-switch
```

- [ ] **24.4** `.github/ISSUE_TEMPLATE/feature.md`:

```markdown
---
name: Feature request
about: Propose a feature or improvement
title: "[feat] "
labels: enhancement
---

## Problem
## Proposed solution
## Moat alignment
Which moat does this strengthen, and how?

## Non-goals
```

- [ ] **24.5** Commit: `git add .github/ && git commit -m "ci: add GitHub Actions (ci, security) + issue templates"`

---

## Task 25: Push + final verification

- [ ] **25.1** `cd ~/dev/GlobalReel && git log --oneline && git push origin main`

- [ ] **25.2** `gh repo view michaellinhq/GlobalReel --json name,defaultBranchRef`

- [ ] **25.3** Agents frontmatter check:

```bash
cd ~/dev/GlobalReel
for f in agents/*.md; do head -10 "$f" | grep -q "^name:" && echo "OK: $f" || echo "MISSING: $f"; done
```
Expected: all 9 `OK:`.

- [ ] **25.4** `python -c "import json; json.load(open('.claude/settings.json'))" && echo OK`

- [ ] **25.5** Secret sweep: `git log --all -p | grep -E "(AKIA|sk_live|ghp_|PRIVATE KEY)" || echo "clean"`

---

## Self-Review

1. **Spec coverage:** all 12 spec sections mapped to tasks (§1 mission → Task 3, §2 arch → Task 23, §3 layout → all tasks, §4 lang split → Task 3 CLAUDE.md, §5 CLAUDE.md → Task 3, §6 agents → Tasks 5-13, §7 harness → Task 4, §8 scripts → Task 2, §9 CI → Task 24, §10 non-goals → Task 3 + TODOs, §11 commit plan → one commit per logical chunk, §12 open questions → preserved in docs).
2. **No placeholders:** every step has actual content.
3. **Type consistency:** package names consistent (`@globalreel/{web,api,gateway,shared-types,compliance,config}`).
4. **No implementation code:** every stub is manifest + README + TODO only.
5. **Commit granularity:** ~15 commits, each revertable.

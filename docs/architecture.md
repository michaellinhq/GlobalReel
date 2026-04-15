# Architecture

Three tiers:

1. **Cloud SaaS** — `apps/web` (Next.js), `apps/api` (NestJS), `apps/bi-worker` (Python).
2. **Local gateway** — `apps/gateway` (Node daemon on Mac Studio). mTLS WebSocket to cloud. BullMQ queues locally.
3. **Hardware swarm** — `apps/automation` (Python + Appium) controlling ~10 Android phones with German consumer SIMs.

Language boundary: TS in cloud + gateway, Python in automation + bi-worker. Cross-boundary: OpenAPI-generated types.

Full architecture: TODO.

# NEXUS-e14

**Agentic Engineer Intelligence & Commerce Decision Platform**

A human-governed GenAI operating system for electronics distribution. NEXUS-e14 sits beside WebSphere Commerce, PIM, OMS, WMS and a ~890k-member engineer community and turns design or sourcing intent into a cited, commercially honest decision package.

[![Status](https://img.shields.io/badge/status-architecture%20spec-0F766E)](#status)
[![Programme](https://img.shields.io/badge/programme-26%20months-1D4ED8)](#programme-facts)
[![Estate](https://img.shields.io/badge/estate-48%20sites%20·%2013%20DCs-334155)](#programme-facts)
[![Integrity](https://img.shields.io/badge/SKU%20invention-0-059669)](#trust-safety-and-cost)
[![ROI frame](https://img.shields.io/badge/ROI%20frame-218%25-7C3AED)](#outcomes)

> Facts are bound. Tools are closed. Money and export require a human. Versions supersede. GPU is optional. Autonomy is earned.

---

## Table of contents

- [Why it exists](#why-it-exists)
- [What shipped](#what-shipped)
- [Platform constitution](#platform-constitution)
- [Architecture](#architecture)
- [Multi-agent orchestration](#multi-agent-orchestration)
- [Model portfolio](#model-portfolio)
- [Knowledge and retrieval](#knowledge-and-retrieval)
- [Canonical APIs](#canonical-apis)
- [Tech stack](#tech-stack)
- [Trust, safety and cost](#trust-safety-and-cost)
- [Service level objectives](#service-level-objectives)
- [Outcomes](#outcomes)
- [Programme facts](#programme-facts)
- [How to read this document](#how-to-read-this-document)
- [Author](#author)

---

## Why it exists

The multi-banner electronics estate already runs commerce: IBM WebSphere Commerce storefronts, a Partner Product Search REST API with HMAC and contract-price fields, BOM upload (capped at 1,000 lines), iBuy approval graphs, cXML/OCI punch-out, DesignLink, RoadTests, and on the order of a million SKUs across 48 localised sites.

What it did not run is a single intelligence layer that can take an engineer’s intent and produce a commercially honest, cited decision package.

| Inherited path | Failure mode |
| --- | --- |
| Lexical search + merchandising boost | Ignores franchise, ATP velocity, datasheet contradiction, accepted community answers |
| FAE email on a Friday 400-line BOM | Hours of latency; lost design-wins |
| Chatbot on the catalogue | Invents order codes and contract prices |
| Tribal alternate knowledge | Second-source recommendations ignore RoadTest contradictions |
| Export class as a datasheet footer | Facets do not capture it; auto-quote is a legal incident |

This is not a workflow problem. Unstructured multimodal input, delayed outcomes (a board still working at 90 days), commercial tail risk, and store-specific entitlement cannot be closed by a rules engine without becoming an unmaintainable expert system.

---

## What shipped

NEXUS-e14 is **not a chatbot bolted onto search**. It is an operating system with a commercial constitution: catalogue intelligence, GraphRAG retrieval, BOM decisioning, iBuy-aware coaching, datasheet QA, community synthesis and RoadTest operations, with a persistent engineer Copilot that drafts and explains under human oversight.

| Domain | Capability | Hard stop |
| --- | --- | --- |
| **Discover** | NL and parametric search, Partner API compatibility, store-scoped ranking | HMAC path never calls an LLM |
| **Understand** | Intent compile, datasheet VLM compile, accepted-answer retrieval | UGC never enters the instruction channel |
| **Match** | BOM 1–1,000 lines; exact then neighbour; DecisionRecords | Unmatched / export → HITL |
| **Alternate** | Typed graph walks: exact, FFF, parametric neighbour, suggested | Walk closure only; export can block |
| **Procure** | Quote freeze via price engine; iBuy coaching; punch-out | Cannot approve or emit a PO |
| **Community** | Accepted-answer-first GraphRAG; injection classifier | Moderator assist only after member accept |
| **Programme** | RoadTest / Design Challenge logistics bound to SKUs | Model cannot select winners |
| **Operate** | LLMOps, eval gates, FinOps, sovereignty pin, designed degrade | Promote blocked on gold-set fail |

A capability that cannot name its hard stop is not a capability; it is a demo.

---

## Platform constitution

Non-negotiable constraints that shape every design:

1. **Price, ATP, tax, contract entitlement and export flags are facts.** The model may narrate or rank; it may not author them.
2. **PIM, price engine, OMS and WMS remain systems of record.** The platform reconciles; it does not overwrite.
3. **Community UGC is retrieved as data.** Instruction isolation is a compile-time property of the context pack, not a prompt suggestion.
4. **One codebase serves a single local site and the full APAC / EMEA / AMER estate.** No fork.
5. **GPU cost is the platform’s risk, not the engineer’s.** Degrade is a product feature.
6. **Export-controlled SKUs and iBuy approval graphs are legal surfaces.** Autonomy stops there.
7. **Partner HMAC and punch-out contracts cannot break** while humans learn to trust citations.

---

## Architecture

Five tiers own request flow. Two planes own control and governance. Cells bound blast radius — a runaway agent loop or a poisoned index affects one cell, not the fleet.

```text
┌─────────────────────────────────────────────────────────────────┐
│  T1  Channel & edge                                             │
│      WCS · Partner HMAC · punch-out · DesignLink · community    │
│      WAF · rate limits · storeInfo.id bound before retrieval    │
├─────────────────────────────────────────────────────────────────┤
│  T2  Experience & API                                           │
│      Command Center · Copilot · BOM · iBuy · GraphQL BFF        │
│      Entitlement claim in every token · no business logic in BFF│
├─────────────────────────────────────────────────────────────────┤
│  T3  Orchestration                                              │
│      Temporal (lifecycle) + LangGraph (specialists) + HITL      │
│      Bounded autonomy · every side effect a typed, idempotent   │
│      tool                                                       │
├─────────────────────────────────────────────────────────────────┤
│  T4  Intelligence                                               │
│      Model gateway/router · vLLM / Ray / Triton · retrieval     │
│      VLM · feature serving · single egress · policy per call    │
├─────────────────────────────────────────────────────────────────┤
│  T5  Systems of record                                          │
│      PIM · price · ATP · OMS · WMS · community object model     │
│      LLM never writes these                                     │
├─────────────────────────────────────────────────────────────────┤
│  Control plane     Eval · routing · flags · FinOps · sovereignty│
│  Governance plane  OPA/Cedar · audit ledger · HITL · ADRs       │
└─────────────────────────────────────────────────────────────────┘
```

### Inquiry flow

1. A channel adapter normalises the payload into an `InquiryEnvelope`, binds `store_id` + actor + entitlement, and appends `inquiry.received` to the store-partitioned event log.
2. A durable `IntentLifecycle` workflow (Temporal) becomes the single writer of lifecycle state.
3. The supervisor graph fans out in parallel to the Safety/Invention Sentinel, the intent classifier and the retrieval planner. **Sentinel is a join barrier:** no money-adjacent node executes until it resolves.
4. The context compiler assembles a minimum-necessary pack. UGC and datasheet text are labelled `DATA`.
5. Specialists propose typed objects (`PartCard`, `BomDecisionRecord[]`, `AlternateWalk`, cited `DatasheetAnswer`) — never free text for a machine to parse as a side effect.
6. Autonomy policy maps each proposal to `AUTO` | `STAGE` | `ESCALATE` | `BLOCK`. Confidence never unlocks a PO.
7. Deterministic services validate and commit. Later outcomes (quote accepted, PO placed, board built, return) join back to the originating DecisionRecord as labels.

### Cell topology

| Cell | Serves | Why split |
| --- | --- | --- |
| APAC | IN / SG / AU / CN-adjacent | Pin, latency, data-residency |
| EMEA | UK / DE / FR / IT / Nordics / JP overlay | Peak Partner traffic + GDPR-adjacent logs |
| AMER | AMER stores | Contract-book isolation from APAC GPU |
| Shared control plane | Gateway policy, model factory, gold, inventory | One promote path; per-store policy documents |

### Isolation (defence in depth)

| Layer | Mechanism |
| --- | --- |
| Identity | Store- and account-scoped OIDC; `store_id` signed into every token and span |
| OLTP | Shared-schema PostgreSQL with RLS on `store_id` |
| Encryption | Per-store data keys; crypto-shred on offboard |
| Event log | Partition key begins with `store_id`; consumers re-filter |
| Vector store | Class-separated collections; store pre-filter before ANN |
| Feature store | Composite keys `store_id#entity_id` |
| Models | Global bases on public + accepted UGC only; no iBuy/contract in SFT |
| Network | Namespace per cell; default-deny; Istio mTLS; egress allow-list |
| Caches | Keys include store, policy version, data class; semantic cache off for commercial class |

---

## Multi-agent orchestration

The agent layer is a **supervised hierarchical multi-agent system inside durable workflows**. It is deliberately not an open-ended group chat.

| Runtime | Owns | Horizon | Technology |
| --- | --- | --- | --- |
| Durable workflow | BOM lifecycle, quote freeze timers, iBuy waits, punch-out sagas, compensation | Seconds to 90 days | Temporal |
| Agent graph | Supervisor routing, specialist execution, parallel fan-out, verify-repair, HITL interrupt | Milliseconds to minutes | LangGraph |

Rule, enforced in code review: anything that must survive a deploy, wait for a human, wait for time, or compensate is a **workflow** concern. Anything that reasons over context to produce a proposal is a **graph** concern.

### Agent inventory

| Agent | Charter | Ceiling |
| --- | --- | --- |
| Supervisor | Classify, plan, merge, request review | A1 — route, never act |
| Invention / Safety Sentinel | SKU regex, price-in-prose, export disguise, UGC-as-instruction | Fail-closed; overrides all |
| Part Search | Hybrid retrieve + explain rank | A0 |
| Datasheet QA | Page-cited answers from compiled DS | No answer without page pin |
| BOM Matcher | Exact then neighbour; emit DecisionRecords | HITL on unmatched / export |
| Alternate Walker | Typed walks + policy sim | Cannot mint SKUs |
| Community Answer | Accepted-answer first | No publish without accept |
| Procurement Copilot | Coach iBuy / freeze; show policy | Zero money side-effects |
| Compliance / Export | Flag and stop | Can only block |
| Verifier (Critic) | UNSUPPORTED_CLAIM, PRICE_IN_PROSE, ORPHAN, POLICY | Veto only; **different model family** |
| Eval / Red-team worker | Offline and shadow | No production side-effects |

### Topology patterns

- **Parallel fan-out with a safety barrier** — Sentinel, Intent and Retrieval-plan run concurrently; Sentinel short-circuits on `INVENTION`, `PRICE_PROSE`, `EXPORT_DISGUISE` or `INJECTION`.
- **Typed handoff** — specialists are invoked through `handoff(target, reason, payload_schema)`, not a message in a transcript.
- **Generate-verify-repair** — at most two repair iterations; then degrade to a deterministic card or a human queue. Unbounded self-reflection is prohibited.

### Budgets

| Guard | Default |
| --- | --- |
| Max graph steps per invocation | 12 |
| Max repair iterations | 2 |
| Token ceiling | 24k in / 2k out |
| Cost ceiling | Per store, per intent — downgrade tier before reject |
| Cycle detection | Same `(node, state-hash)` twice → hard stop |

### Tool fabric

Internal capabilities are MCP servers behind a Go tool gateway (authz, capability tokens, rate limits, idempotency, data-class egress, audit). A2A is used **only** at the organisational boundary (manufacturer agents, 3PL, punch-out partners).

OMS emit is a workflow activity after a human token. The LLM never calls it.

---

## Model portfolio

Foundation models are commodities behind a gateway. Proprietary performance comes from families trained on the platform’s own decision and outcome data.

| ID | Model | Type | Serve |
| --- | --- | --- | --- |
| **M1** | Intent + slot encoder | Multi-task encoder | Triton p95 ≤ 60 ms |
| **M2** | Invention / injection sentinel | Classifier ensemble + rules | Triton dedicated, p95 ≤ 150 ms |
| **M3** | Engineer voice adapters | LoRA on open-weight instruct | vLLM multi-LoRA |
| **M4** | SKU / BOM ranker | LambdaRank over feature vectors | Triton FIL p95 ≤ 25 ms |
| **M5** | Quote-conversion / risk | GBDT + survival | Batch + online |
| **M6** | Demand / kit forecaster | Hierarchical GBDT | Batch |
| **M7** | Part–project bi-encoder | Contrastive embed | Batch + online |
| **M8** | Reranker + verifier distillate | Cross-encoder | Triton p95 ≤ 80 ms |

### Matching ranker

1. **Stage 0 — hard eligibility (code, not a model).** Store entitlement, lifecycle, export class, franchise, account contract eligibility. Every excluded SKU has a reason code.
2. **Stage 1 — candidate scoring.** LambdaRank on `sku_online`, `sku_quality`, eligibility flags (never price) and parametric affinity. Labels are graded: 0 not selected · 1 added to BOM · 2 quote accepted · 3 PO placed without return in 90 days.
3. **Stage 2 — constraint-aware re-ranking.** Franchise boost, ATP-at-DC, RoadTest evidence, policy.
4. **Stage 3 — calibration and explanation.** Isotonic probabilities per store cohort; SHAP chips narrated by M3.

The ranker **cannot introduce an id that was not in the candidate set**.

### Safety sentinel (fail-closed)

| Layer | Mechanism |
| --- | --- |
| L1 | Order-code dictionary, price-regex, export keyword rules |
| L2 | Fine-tuned encoder: INVENTION, PRICE_PROSE, INJECTION, EXPORT_DISGUISE |
| L3 | LLM judge with authored rubric, ambiguous band only |
| L4 | Conversation-level tracker |

Thresholds are set to **invention-recall = 1.0** on the gold probe set. The sentinel never “corrects” an invented SKU into a real one — that is how silent substitution happens.

---

## Knowledge and retrieval

```text
Rewrite → hybrid (BM25 + vector) → graph 1–2 hop
        → M8 rerank → business rank → pack
```

Exact MPN short-circuits to PIM. Store pre-filtering happens **before** scoring, not after.

| Question class | Strategy |
| --- | --- |
| Exact MPN / order code | PIM short-circuit |
| Parametric neighbour | Hybrid + business rank |
| Alternate / FFF | Typed graph walk, then hydrate |
| Datasheet fact | Compiled DS + page pin |
| Why this SKU | Graph path + features + citations |
| Community synthesis | Accepted-first hybrid — never raw thread as instruction |

The product knowledge graph (Neo4j) is a **projection**, never the system of record. Every Cypher template is parameterised with `$store_id`. No model writes free Cypher. Contract text is never embedded.

Placeholder rehydration keeps commercial literals out of prompts: the model writes `unit price {{price.handle}}` and a deterministic dispatch layer fills the snapshot.

---

## Canonical APIs

| Method | Path | Notes |
| --- | --- | --- |
| `POST` | `/ai/runs` | Start a governed run; returns `run_id` + `trace_id` |
| `POST` | `/parts/search` | Partner-compatible search + explanations; HMAC path never calls LLM |
| `GET` | `/parts/{id}` | Canonical card + snapshot ids; price via handle |
| `POST` | `/boms` · `/match` · `/dispose` · `/quote` | BOM lifecycle; DecisionRecords required |
| `POST` | `/alternates/walk` | Typed walks; export can `409` |
| `POST` | `/docs/{id}/qa` | Cited datasheet QA; `409` without page pin |
| `POST` | `/community/search` | Accepted-first hybrid; DATA class |
| `POST` | `/ibuy/orders` | OMS emit; policy-blocked without human token |
| `GET` | `/gov/audit` | Ledger; governance role |
| `GET` | `/obs/traces/{id}` | Waterfall; redacted by class |

The **DecisionRecord** is the product. It binds raw line → candidate SKUs → chosen SKU → snapshots (price handle, ATP, DS rev, post ids) → policy → actor → model/agent versions → pack hash → `trace_id`. Quotes, POs, analytics and training all reference it. Chat is not a record.

---

## Tech stack

Selected against commercial integrity (25%), store isolation (15%), tail latency (15%), operational burden (15%), portability (10%), unit cost (10%) and ecosystem maturity (10%). AWS-primary for the cell estate; Azure as provider-independent secondary for inference failover.

| Layer | Decision |
| --- | --- |
| Cloud | AWS multi-account landing zone; Azure AI Foundry for failover |
| Compute | EKS per cell, Karpenter, KEDA, Istio mTLS, default-deny NetworkPolicy |
| Experience | TypeScript GraphQL BFF; Python FastAPI + Pydantic; Go tool gateway |
| Orchestration | Temporal + LangGraph (graphs as Temporal activities) + MCP |
| AI control plane | LiteLLM-core gateway, OPA/Cedar, data-class routing |
| Foundation models | Bedrock primary; Azure AI Foundry secondary; capability-class binding |
| Serving | Triton (M1/M2/M4/M8); vLLM multi-LoRA (M3) |
| Data | Aurora PostgreSQL + RLS; MSK; Flink; Iceberg on S3 |
| Features | Feast; Redis/Dynamo online; Iceberg offline; PIT joins |
| Retrieval | OpenSearch BM25 + pgvector / k-NN; Neo4j GraphRAG |
| Training | SageMaker / Ray; PEFT; LightGBM; MLflow |
| Trust | PyRIT, Giskard, garak, promptfoo; domain attack corpora in-house |
| Observability | OpenTelemetry GenAI conventions; Grafana; Langfuse |
| Delivery | Terraform, GitHub Actions (OIDC), Argo CD, Argo Rollouts, ADRs |

### Deliberately not selected

- A site-wide SaaS copilot with implicit web retrieval
- Letting the LLM call SQL or the price table
- A single shared vector index for all data classes
- Replacing WCS checkout in v1
- Training on raw iBuy payloads to “personalise”
- Free-form multi-agent conversation on the PO path

---

## Trust, safety and cost

Trust, safety and cost are governed as **one system** because they fail together.

### Threat model (OWASP LLM Top 10, specialised)

| Threat | Primary controls |
| --- | --- |
| Direct prompt injection | Instruction hierarchy, injection classifier, persona-scoped tools, verifier veto |
| Indirect injection (UGC, datasheets, punch-out) | DATA-labelled delimited blocks; document-reading nodes have no money tools |
| Sensitive disclosure | Store-keyed caches, pre-filtered retrieval, store-confined adapters |
| Excessive agency | Autonomy policy external to agents; OMS dispatch only after human token |
| Improper output handling | Typed outputs only; no model output reaches a query engine unescaped |
| Unbounded consumption | Runtime budgets, per-store quotas, spend-velocity alerts |
| Silent substitution | Sentinel never remaps; unmatched stays unmatched |

### Injection defence in depth

Channel normalisation → classification → labelled DATA blocks → architectural separation of document-reading and money tools → capability tokens bound to `bom_id` / `intent_id` → Istio egress allow-list → different-family verifier → store-level alerting and automatic autonomy demotion.

### Release gates that do not slip

| Gate | Evidence |
| --- | --- |
| Safety / invention | Invention recall = 1.0 on gold; zero critical red-team successes |
| Commercial integrity | Price-in-prose = 0; no PO without DecisionRecords |
| Privacy / tenancy | Automated cross-store leakage tests |
| Quality | Node, trajectory and outcome eval at or above champion |
| Reliability | SLOs, failover tested, degrade modes demonstrated |
| Cost | Cost per accepted BOM within plan at planning load |

Kill switches exist at fleet, store, intent and endpoint. Autonomy is earned per store and per intent and is **automatically withdrawn** when orphan-rate or HITL-reject-rate breaches its control limit.

---

## Service level objectives

| Objective | Target |
| --- | --- |
| Search P95 (deterministic + hybrid) | ≤ 300 ms at the edge |
| GraphRAG cited card P95 | ≤ 1.2 s |
| Copilot first visible token | P50 ≤ 0.9 s, P95 ≤ 1.8 s |
| BOM 100-line match P95 | ≤ 8 s |
| BOM 400-line industrial P95 | ≤ 20 s |
| SKU-invention rate on gold | **0** |
| Price-invention rate on gold | **0** |
| Citation orphan rate | < 1% of asserted SKU claims |
| Injection follow on labelled UGC | **0** |
| Isolation | Zero cross-store contract-book exposure |
| Continuity | If the GPU pool dies, search, ATP, price and checkout remain available |

### First-token budget

| Segment | Budget |
| --- | --- |
| Edge + BFF authz + store bind | ≤ 40 ms |
| Intent classify + policy (M1, not an LLM) | ≤ 60 ms |
| Deterministic search / ATP / price tools | ≤ 120 ms combined P95 |
| Hybrid retrieve if required | ≤ 250 ms |
| Reasoner first token if required | ≤ 400 ms (hedged after p90) |

### Continuity

| Capability | RTO | RPO |
| --- | --- | --- |
| Deterministic search / ATP / price / checkout | minutes | commerce SoR |
| GraphRAG / Copilot | hours on degrade path | rebuild from CDC |
| Eval / training | days | artifact registry |
| Audit ledger | minutes | zero acceptable loss |

Degrade modes: cited-card-without-reasoner, lexical-only search, human queue. Game days cover GPU loss, community 403s, HMAC rotation, poisoned UGC and Partner burst.

---

## Outcomes

Figures are the programme narrative across flagged stores.

| Outcome | Before | After |
| --- | --- | --- |
| Time-to-quote on 400-line industrial BOM | Multi-hour FAE email loop | Seconds to DecisionRecords; HITL only on the tail |
| Search explanation | Lexical + merchandising boost | Cited card with franchise / ATP / accepted-answer chips |
| Partner Search API | At risk in any rewrite | HMAC path unchanged; explanation fields additive |
| Checkout during GPU events | Would couple if Copilot owned cart | Degrade banner; WCS checkout unaffected |
| SKU invention into PO | Unmeasured (prototypes invented) | 0 on gold and production monitors |
| Citation orphan rate | n/a | < 1% of asserted SKU claims |
| Programme narrative ROI | Inherited email-and-facet baseline | **218%** on the NEXUS-e14 frame |

### Platform economics

- ~70% of hot-path invocations served by encoders, rankers and store adapters rather than frontier models
- 45–60% reduction in billable input tokens from prefix caching and context discipline versus naive RAG
- Cost per accepted BOM held within plan through fleet growth; attributable per store

The durable advantage is not the language model. It is the closed loop: a decision is made with reproducible features, a human edits or overrides it, the outcome arrives days later, the three are joined into a labelled DecisionRecord, and the next model — ranker, adapter, sentinel, autonomy policy — is measurably better for that store.

---

## Programme facts

| Fact | Value |
| --- | --- |
| Estate | APAC · EMEA/JP · AMER multi-banner stores · community portal |
| Scale | ~890k members · 48 local sites · 13 DCs · 1.0–1.3M SKUs · 2,000+ manufacturers |
| Duration | 26-month platform transformation |
| Organisation | 23-member cross-functional team across seven squads |
| Accountable role | Principal AI Architect — AI control plane, standing veto, release gates |
| Specification date | 28 September 2026 |

### Team

| Team | Size | Owns |
| --- | --- | --- |
| AI Platform | 4 | Gateway, router, context compiler, guardrails, persona profiles |
| Agent Engineering | 4 | Supervisor, specialists, graph definitions, trajectory evaluation |
| ML Platform | 3 | Feature platform, training pipeline, Triton / vLLM serving |
| Applied ML | 3 | M1–M8, experiments, fairness / locale analysis |
| Data and Graph | 3 | Event log, lakehouse, CDC, lineage, Neo4j projection |
| Adapters | 3 | WCS, Partner API, punch-out, community anti-corruption, DesignLink |
| SRE / Eval | 3 | SLOs, mesh, IaC, CI/CD, chaos, DR, gold sets, red-team automation |

---

## How to read this document

This README is the control-plane constitution for a GenAI operating layer that sits **beside** a living commerce and community estate. It is not a claim that the live public website already runs LangGraph.

The live estate still presents WCS storefronts, a Partner HMAC API, BOM upload, iBuy, punch-out and a bot-hostile community. Those surfaces are the systems of record and the adapters. NEXUS-e14 is the intelligence layer designed so those surfaces do not have to be rewritten for an engineer to get a cited, commercially honest decision package.

Two review checks:

1. If you cannot point to the WCS path, the Partner field, the BOM cap, the 13 DCs or the 48 sites behind a given module, that module is decoration.
2. If you cannot point to the hard stop behind a given agent, that agent is a demo.

The finished platform must feel like an engineer operating environment with a GenAI control plane — not a distributor website with a chatbot attached.

---

## Status

Architecture specification · September 2026 · companion artefacts (backend module catalogue, live UI/UX specification) are the surface grammar this constitution requires those surfaces to obey.

This repository documents the Principal AI Architecture. Prompts, policies, graphs, adapters and models ship through the same promotion rules as code: eval digest, ADR, and gold-set gates.

---

## Author

**Jyotirmoy Bardhan**  
Principal AI Architect  
26-month platform transformation · 23-member engineering organisation  
September 2026

---

## License

Proprietary. Internal architecture specification. Not an open-source release.

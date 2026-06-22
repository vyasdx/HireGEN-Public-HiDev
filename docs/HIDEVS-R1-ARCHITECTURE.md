# HireGEN — HiDevs x Mastra Hackathon · Round-1 Architecture

**Track:** AI Hiring Copilot
**Team:** Vedavyas Vayalpadu
**Issue:** ENH-HIREGEN-040
**Date:** 2026-06-21
**Round-1 deadline:** 2026-07-01
**Round-1 artifacts:** PRD + this full-stack architecture document
**Mandatory stack:** Mastra orchestration/HITL + Qdrant memory/retrieval + Enkrypt safety/eval
**Language rule:** TypeScript end-to-end
**Official resource set used:** Mastra docs/agents/workflows/examples, Qdrant documentation, Enkrypt AI/Rayder/Skill Sentinel/Academy resources provided by HiDevs.

> Round 1 is a **design submission**, not a runnable agent submission. The working agent is for the July 4-11 build sprint and July 12 finale. This architecture is written so the build sprint can start from an approved, sponsor-aligned plan without rethinking the system.

---

## 0. Executive Architecture Bet

HireGEN is not a greenfield hackathon idea. It is an existing proof-of-capability hiring MVP being re-platformed into a TypeScript agent system:

- Existing `skill-graph.ts` becomes a Mastra skill-graph tool.
- Existing `role-intelligence.ts` becomes the seed for Qdrant role memory.
- Existing L0-L4 evidence labels become Enkrypt grounding gates.
- Existing recruiter dashboard becomes the human-in-the-loop decision surface.
- Existing proof profile and target-fit workflows become the input/output contract for the agent.

**Product thesis:** HireGEN should be the proof layer for hiring. The agent should not merely rank resumes; it should explain what is proven, what is inferred, what is missing, and which role-specific validation task should be run before a recruiter trusts the match.

---

## 1. Challenge Capability Map

| Required capability | HireGEN design response | Primary stack owner |
|---|---|---|
| Analyze job descriptions | Parse JD into role, seniority, must-have skills, proof bar, interview topics | Mastra |
| Semantically rank candidates | Match skill graphs against JD, Qdrant role context, and evidence vectors | Mastra + Qdrant |
| Generate role-specific interviews | Produce questions and lab tasks from candidate gaps and target role | Mastra + Qdrant |
| Evaluate responses | Grade answers and lab outputs against rubric and retrieved evidence | Mastra + Enkrypt |
| Recruiter-ready recommendations | Return Shortlist / Needs Lab / Reject for Now with reasons | Mastra HITL |
| Ensure fairness and safety | Bias, grounding, toxicity, and PII gates before recruiter display | Enkrypt |

The five verbs from the prompt map cleanly:

| Verb | HireGEN behavior |
|---|---|
| Think | Build skill graph, score role fit, reason over direct proof vs logical transfer |
| Retrieve | Pull role templates, similar candidates, evidence snippets, and prior outcomes from Qdrant |
| Remember | Store recruiter decisions, validation outcomes, and candidate proof history in Qdrant |
| Evaluate | Score interview/lab responses and run Enkrypt safety/eval gates |
| Act | Recommend recruiter action and persist the decision after HITL review |

---

## 2. End-to-End Agent Workflow

### 2.1 Mastra workflow

```text
ingest(JD + candidates)
  -> buildSkillGraph
  -> retrieveRoleContext(Qdrant)
  -> matchAndRank
  -> generateInterview
  -> evaluateResponses
  -> safetyGate(Enkrypt)
  -> HITL recruiter decision
  -> recommendation
  -> persistOutcome(Qdrant)
```

### 2.2 Workflow details

| Step | Mastra primitive | Input | Output |
|---|---|---|---|
| `ingest` | Workflow step + validation tool | JD text, resumes, GitHub URLs, project URLs | Normalized candidate and role payloads |
| `buildSkillGraph` | Agent/tool wrapping existing HireGEN engine | Candidate evidence | Skills, badges, gaps, evidence levels, source refs |
| `retrieveRoleContext` | Qdrant search tool | Role brief + extracted requirements | Nearest role templates, aliases, proof bar, interview rubric |
| `matchAndRank` | Ranking agent | Skill graph + role context | Ranked candidates with score breakdown |
| `generateInterview` | Interview agent | Target role + candidate gaps | Role-specific questions + validation-lab task |
| `evaluateResponses` | Evaluation agent | Candidate answers/lab output + rubric | Response score, evidence score, concern flags |
| `safetyGate` | Enkrypt eval tool | Ranking, interview, evaluation, recommendation | PASS / BLOCK / NEEDS_REVIEW with reasons |
| `HITL recruiter decision` | Mastra suspend/resume | Safety-gated recommendation | Shortlist / Needs Lab / Reject for Now |
| `recommendation` | Output formatter | Recruiter decision + candidate proof | Recruiter-ready candidate card/report |
| `persistOutcome` | Qdrant upsert tool | Decision and evidence outcome | Persistent recruiter/candidate memory |

### 2.3 Why interview generation and response evaluation matter

Ranking alone is not enough for this track. HireGEN's standout workflow is:

1. Identify a candidate who is promising but not fully proven.
2. Generate a role-specific interview or validation-lab task targeted to the exact missing proof.
3. Evaluate the response against a rubric.
4. Run Enkrypt gates before presenting the result.
5. Let the recruiter decide.

This maps directly to HireGEN's planned Validation Lab: timed, paste-resistant, role-fit proof tasks for gaps such as "RAG over incident runbooks," "AIX/Linux troubleshooting," "backend API debugging," or "healthcare BA requirement analysis."

---

## 3. Mastra Orchestration Layer

Mastra is the control plane for all agent behavior.

Design uses the official Mastra concepts from the provided docs: agents for goal-directed reasoning, tools for external actions, workflows for multi-step orchestration, and HITL suspend/resume for recruiter decisions. The build-sprint implementation should stay close to Mastra examples rather than inventing a custom agent runner.

### 3.1 Agents

| Agent | Purpose | Tools |
|---|---|---|
| `jdAnalyzerAgent` | Extracts role intent, seniority, must-have skills, nice-to-haves, proof bar | `roleParserTool`, `qdrantSearchTool` |
| `skillGraphAgent` | Converts candidate evidence into grounded skill graph | `skillGraphTool`, `githubScanTool`, `projectProofTool` |
| `rankerAgent` | Scores candidates with direct proof, logical transfer, missing proof, and confidence | `qdrantSearchTool`, `scoreBreakdownTool` |
| `interviewAgent` | Generates role-specific interview questions and lab tasks | `rubricTool`, `qdrantSearchTool` |
| `responseEvalAgent` | Evaluates answers/lab outputs against rubric and evidence | `rubricTool`, `enkryptEvalTool` |
| `recruiterCopilotAgent` | Produces final recruiter recommendation and next action | `enkryptEvalTool`, `qdrantUpsertTool` |

### 3.2 Tools

| Tool | Existing HireGEN source | Build-sprint implementation |
|---|---|---|
| `skillGraphTool` | `web/src/lib/skill-graph.ts` | Port current skill extraction as a Mastra tool |
| `roleIntelligenceTool` | `web/src/lib/role-intelligence.ts` | Seed Qdrant and retrieve role context |
| `githubScanTool` | Existing GitHub/project scan logic | Repo metadata, README quality, live URL signal |
| `projectProofTool` | Product Proof fields | Validate live product URL and candidate-stated contribution |
| `qdrantSearchTool` | New | Search role, candidate, evidence, and outcome collections |
| `qdrantUpsertTool` | New | Persist vectors and outcome memory |
| `enkryptEvalTool` | New | Bias, grounding, toxicity, PII, and response-quality gates |
| `rubricTool` | New | Role-specific question/lab scoring rubric |

### 3.3 Human in the loop

The agent must never make final hiring decisions. Mastra pauses after Enkrypt safety checks and presents the recruiter with:

- candidate rank and score breakdown,
- direct proof,
- logical transfer,
- missing proof,
- generated interview/lab result if available,
- recommended action.

The recruiter then chooses:

- `Shortlist`
- `Needs Lab`
- `Reject for Now`
- `Keep Warm`

That decision is persisted as memory for future matching.

---

## 4. Qdrant Memory And Retrieval Layer

Qdrant is both semantic retrieval and long-term memory.

Design uses Qdrant as a vector database with collections, payload metadata, semantic search, and filterable retrieval. Candidate matching should combine semantic similarity with payload filters such as experience band, location preference, evidence level, interview status, and recruiter outcome.

| Collection | Stores | Used by |
|---|---|---|
| `role_intel` | Role templates, required skills, aliases, logical transfers, proof templates | JD analysis, interview generation, ranking |
| `candidates` | Candidate-level profile embeddings and skill-graph summaries | Ranking, dedupe, similar candidate search |
| `evidence` | Resume snippets, GitHub/project evidence, source refs, evidence level | Grounding checks, profile explanation, Enkrypt hallucination gate |
| `outcomes` | Recruiter decisions, interview/lab results, reasons, timestamps | Memory, recruiter preference learning, future ranking calibration |

### 4.1 Retrieval design

- JD text is embedded and searched against `role_intel`.
- Candidate skill graph is embedded and searched against `role_intel` and `candidates`.
- Every recruiter-facing claim must retrieve supporting evidence from `evidence`.
- Interview generation retrieves missing skills, role rubrics, and similar past validation tasks.
- Outcome memory retrieves how the recruiter previously handled similar profiles.

### 4.2 Why Qdrant is not just storage

Qdrant is the mechanism that prevents the copilot from becoming a generic resume chatbot. It gives the agent:

- semantic role understanding,
- evidence-grounded explanations,
- persistent recruiter memory,
- cross-session improvement,
- repeatable retrieval for safety evaluation.

---

## 5. Enkrypt Safety And Evaluation Layer

Enkrypt is the hard gate before any ranking, evaluation, or recommendation reaches the recruiter.

Design uses Enkrypt as the safety/evaluation layer for agent outputs: hallucination and grounding checks, bias and fairness checks, toxicity/safety checks, PII risk checks, and secure agent behavior review. Enkrypt tools such as Rayder and Skill Sentinel are treated as sponsor-aligned references for red-team style evaluation and secure AI coding/eval discipline.

| Gate | What it checks | Result |
|---|---|---|
| Grounding | Is each skill claim supported by retrieved evidence? | Demote or block unsupported claims |
| Bias/fairness | Does rank change when name/city/college proxies change but evidence is constant? | Block biased recommendation |
| Toxicity/safety | Is candidate feedback respectful and safe? | Rewrite or block |
| PII/DPDP | Is sensitive candidate data exposed beyond intended scope? | Redact or block |
| Interview quality | Are questions role-specific and non-discriminatory? | Regenerate or mark for review |
| Response-eval quality | Is the response score based on rubric evidence, not style/pedigree? | Block unsupported evaluation |

### 5.1 Enkrypt as differentiator

Most hiring AI demos stop at ranking. HireGEN's position is stronger: **ranking is not trusted until it passes a fairness and grounding gate**. Hiring is a high-stakes domain; a biased or hallucinated rank is not a minor bug, it is product failure.

The Enkrypt golden set becomes a permanent HireGEN regression suite:

- fairness pairs with swapped name/city/college,
- unsupported-claim traps,
- role-transition profiles,
- strong private-work candidates with weak public GitHub,
- student profiles where project proof matters more than work history.

This also strengthens HireGEN's ADDA/anti-drift discipline because safety failures become repeatable tests, not anecdotes.

---

## 6. Full-Stack Architecture

### 6.1 Frontend

| Surface | Purpose |
|---|---|
| Recruiter dashboard | Upload JD/candidates, inspect ranked shortlist, trigger interview/lab, take HITL decision |
| Candidate proof profile | Shows skill graph, proof badges, gaps, roadmap, lab result |
| Validation Lab screen | Candidate answers role-specific questions and completes timed task |
| Admin/eval view | Shows Enkrypt verdicts, failed gates, audit trail, Qdrant memory status |

Frontend stack:

- Next.js 16
- TypeScript
- Tailwind CSS
- Existing HireGEN UI primitives and Cobalt/Sky design system

### 6.2 Backend

Backend is a TypeScript service layer that exposes app APIs and runs the Mastra workflow.

| Backend component | Responsibility |
|---|---|
| Next.js API routes | Uploads, profile generation, recruiter actions |
| Mastra service | Agent workflow orchestration |
| Evidence parser | Resume/project/GitHub text extraction |
| Safety/eval adapter | Enkrypt API calls and verdict normalization |
| Vector adapter | Qdrant search/upsert |
| Persistence adapter | Stores profile/run metadata and audit logs |

### 6.3 APIs

| API | Method | Purpose |
|---|---|---|
| `/api/hidevs/runs` | `POST` | Start JD + candidate agent run |
| `/api/hidevs/runs/[id]` | `GET` | Read run status and recommendation output |
| `/api/hidevs/interviews` | `POST` | Generate interview/lab for selected candidate |
| `/api/hidevs/evaluations` | `POST` | Submit response/lab output for evaluation |
| `/api/hidevs/recruiter-decision` | `POST` | Resume Mastra HITL workflow with recruiter decision |
| `/api/hidevs/audit/[runId]` | `GET` | Fetch Enkrypt verdicts and evidence trace |

### 6.4 Databases and storage

| Store | Purpose |
|---|---|
| Qdrant Cloud | Role memory, candidates, evidence, outcomes |
| Neon PostgreSQL | Run metadata, user/session records, profile IDs, audit references |
| Cloudflare R2 | Uploaded resumes, generated reports, lab artifacts |
| Vercel KV or Postgres table | Short-lived run/session state if needed |

### 6.5 Integrations

| Integration | Purpose |
|---|---|
| Mastra | Workflow, tools, model routing, HITL |
| Qdrant | Vector memory and retrieval |
| Enkrypt AI | Safety/eval gates |
| GitHub API | Repo metadata, README, language mix, contribution signals |
| LLM provider via Mastra | Reasoning and structured generation |
| Clerk, later | Recruiter/candidate authentication |
| Vercel | Web hosting and API deployment |

### 6.6 Deployment architecture

Round 1 is design-only. Build-sprint deployment target:

```text
Browser
  -> Vercel Next.js frontend
  -> Next.js API routes
  -> Mastra TypeScript workflow service
  -> Qdrant Cloud
  -> Enkrypt AI
  -> Neon PostgreSQL
  -> Cloudflare R2
  -> GitHub API / LLM provider
```

For the build sprint, the Mastra service can run:

- inside Next.js API routes for the smallest deployable version, or
- as a separate TypeScript worker/service if long-running HITL workflows require it.

The demo path should prefer the smallest reliable version: one JD, a small candidate set, one generated interview, one evaluated response, one recruiter decision.

---

## 7. Model And Provider Strategy

Current HireGEN MVP runtime uses OpenAI through `OPENAI_MODEL`, defaulting to `gpt-4o-mini`.

For the HiDevs Mastra design:

- Mastra is provider-agnostic.
- Default high-reasoning model for the build sprint: `claude-sonnet-4-6`.
- Cheap/background steps can use `claude-haiku-4-5`.
- Existing OpenAI adapter remains a fallback because the MVP already works with OpenAI structured output.

No retired Claude Sonnet 4 model ID is used. This aligns with `DEC-HIREGEN-005`.

---

## 8. Round-1 Scope Versus Build Sprint

### Round 1: by 2026-07-01

Deliver only:

- PRD covering problem, target users, solution approach, key features, journey, expected outcomes.
- Full-stack architecture covering agent workflows, Mastra, Qdrant, Enkrypt, frontend, backend, APIs, databases, integrations, deployment.

No runnable skeleton is required for Round 1.

### Build sprint: 2026-07-04 to 2026-07-11

Build:

- Mastra workflow for one recruiter run.
- Qdrant `role_intel` and `evidence` collections seeded.
- Candidate ranking over a small dataset.
- Interview/lab generation.
- Response evaluation.
- Enkrypt grounding gate and one fairness check.
- Recruiter HITL action.
- Persisted outcome.

### Finale: 2026-07-12

Demo:

- JD analysis,
- ranked candidates,
- role-specific interview/lab,
- response evaluation,
- Enkrypt safety verdict,
- recruiter-ready recommendation,
- persisted memory.

---

## 9. Rubric Self-Map

| Judging axis | Weight | Architecture response |
|---|---:|---|
| Mastra integration depth | 25% | Multi-agent workflow, tools, branching, HITL suspend/resume, provider routing |
| Qdrant integration quality | 20% | Four collections used for role context, evidence grounding, candidate memory, outcomes |
| Enkrypt AI coverage | 20% | Hard gate for grounding, bias, toxicity, PII, interview quality, response evaluation |
| Agent output quality | 20% | Recruiter-ready recommendation, interview/lab, evaluation, next action |
| Impact and novelty | 15% | Proof-not-pedigree hiring, Tier-2/3 visibility, fairness as a hard gate |

---

## 10. Risks And Mitigations

| Risk | Mitigation |
|---|---|
| Architecture overreach before July 1 | Keep Round 1 docs clear; build only after shortlist |
| Enkrypt API latency or quota | Cache verdicts by evidence hash; fail closed to manual review |
| Qdrant schema drift | Version collection payload schema and seed from `role-intelligence.ts` |
| Bias eval too narrow | Start with name/city/college proxy pairs; expand golden set after pilot feedback |
| Recruiter distrusts score | Always show "Why this score?" and evidence labels |
| Candidate with private/enterprise work is under-ranked | Product proof, case-study proof, private review, and lab proof are first-class evidence paths |
| Long-running HITL workflow complexity | Start with single run/session; persist state in Postgres and resume via recruiter action |

---

## 11. Build-Sprint File Plan

Proposed TypeScript structure after Round 1 shortlist:

```text
web/src/agents/hidevs/
  hiring-copilot.workflow.ts
  agents/
    jd-analyzer.agent.ts
    skill-graph.agent.ts
    ranker.agent.ts
    interview.agent.ts
    response-eval.agent.ts
    recruiter-copilot.agent.ts
  tools/
    skill-graph.tool.ts
    qdrant.tool.ts
    enkrypt-eval.tool.ts
    rubric.tool.ts
  schemas/
    hidevs-run.schema.ts
    recommendation.schema.ts
  evals/
    fairness-golden-set.ts
    grounding-cases.ts
```

This keeps the agent implementation near the existing Next.js codebase while remaining separable if Mastra needs a standalone service later.

---

## 12. Submission Checklist

- [x] Track selected: AI Hiring Copilot
- [x] PRD drafted
- [x] Architecture doc corrected to Round-1 design-only scope
- [x] Mastra workflow includes ranking, interview generation, response evaluation, HITL, and persistence
- [x] Qdrant collections defined
- [x] Enkrypt hard gates defined
- [x] Frontend/backend/API/database/integration/deployment layers included
- [ ] Final Vyas review
- [ ] Submit PRD + architecture before 2026-07-01

---

HireGEN · Get hired on proof, not pedigree.

## 13. Official Source Links

- Mastra documentation: https://mastra.ai/docs
- Mastra agents overview: https://mastra.ai/docs/agents/overview
- Mastra workflows overview: https://mastra.ai/docs/workflows/overview
- Mastra GitHub: https://github.com/mastra-ai/mastra
- Mastra examples: https://github.com/mastra-ai/mastra/tree/main/examples
- Qdrant documentation: https://qdrant.tech/documentation/
- Enkrypt AI: https://www.enkryptai.com/
- Enkrypt secure vibe coding: https://www.enkryptai.com/solutions/secure-vibe-coding
- Enkrypt Skill Sentinel: https://github.com/enkryptai/skill-sentinel
- Enkrypt Rayder: https://www.enkryptai.com/rayder
- Enkrypt Rayder GitHub: https://github.com/enkryptai/rayder
- Enkrypt Academy: https://academy.enkryptai.com/

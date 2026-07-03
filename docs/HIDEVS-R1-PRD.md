# HireGEN — HiDevs × Mastra Hackathon · Round-1 PRD

**Track:** AI Hiring Copilot · **Team:** Vedavyas Vayalpadu · **Date:** 2026-06-21
**Round-1 artifacts:** this PRD + [`HIDEVS-R1-ARCHITECTURE.md`](HIDEVS-R1-ARCHITECTURE.md) · **Issue:** ENH-HIREGEN-040
**Mandatory stack:** Mastra (orchestration/HITL) · Qdrant (memory/RAG) · Enkrypt AI (safety/eval) — TypeScript-only.

**Round-1 scope:** design submission only — PRD + full-stack architecture by 2026-07-01. The working Mastra/Qdrant/Enkrypt agent is planned for the 2026-07-04 to 2026-07-11 build sprint and 2026-07-12 finale.

---

## 1. Problem

Hiring in India is broken at the **first pass**, in two directions at once:

- **Recruiters drown in resume noise.** A single role draws hundreds of look-alike resumes optimized for keywords, not capability. First-pass screening is manual, slow, and judges *pedigree* (college, company brand, network) because real signal is expensive to assess.
- **Proven candidates get filtered out.** Tier-2/3 talent with genuine proof — shipped projects, public GitHub, live products — is structurally invisible to network-biased platforms. The people most able to *do the job* are the least able to *get seen*.

And the AI "fix" most teams reach for makes it worse: an LLM that ranks resumes will happily **hallucinate skills** and **amplify bias** (name, gender, city, college as proxies) — in the single highest-stakes domain where that is unacceptable.

**The gap:** there is no hiring copilot that is simultaneously (a) **proof-grounded** (every claim traces to evidence), (b) **fair** (bias-audited before a human sees it), and (c) **end-to-end** (JD → ranking → interview → evaluation → recommendation).

## 2. Target users

| User | Pain | What HireGEN gives them |
|---|---|---|
| **Recruiter / hiring manager** (primary — Indian SMBs, startups, agencies) | Hundreds of resumes, no time, no reliable signal, rising fairness/DPDP scrutiny | A ranked, **evidence-backed, bias-checked** shortlist with reasons + a next action per candidate |
| **Candidate** (secondary — esp. Tier-2/3, non-pedigree, career-switchers) | Real proof exists but gets filtered before a human looks | A skill graph that makes proof legible + a gap plan to close the rest |
| **Talent-ops / compliance lead** (tertiary) | Needs auditable, defensible, non-discriminatory screening | An Enkrypt-gated pipeline with a fairness + grounding audit trail |

## 3. Solution approach

**An AI Hiring Copilot that hires on proof, not pedigree** — a Mastra-orchestrated agent that:
1. understands a **job description / role brief**,
2. builds a **verifiable skill graph** for each candidate from GitHub + resume + live projects (with L0–L4 evidence levels),
3. **semantically ranks** candidates against the role via Qdrant,
4. **generates a role-specific interview** and **evaluates the candidate's responses** (incl. a timed Validation-Lab task),
5. runs every output through an **Enkrypt safety gate** (bias, hallucination, toxicity) *before a recruiter sees it*,
6. produces a **recruiter-ready recommendation** (Shortlist / Needs-Lab / Reject-for-now + reason + top gap + plan), with the decision **remembered** across sessions.

The five verbs the challenge demands — **Think · Retrieve · Remember · Evaluate · Act** — map 1:1 (see Architecture §1).

## 4. Key features

| # | Feature | Required capability it satisfies | Stack |
|---|---|---|---|
| F1 | **JD / role-brief understanding** — parse a free-text role into required skills, seniority, proof bar | "analyzes job descriptions" | Mastra + Qdrant `role_intel` |
| F2 | **Proof skill-graph** — resume + multi-GitHub + live-project → skills with evidence refs + L0–L4 level | proof grounding (HireGEN wedge) | Mastra tool (ports `skill-graph.ts`) |
| F3 | **Semantic candidate ranking** — direct match / logical transfer / missing proof, scored | "semantically ranks candidates" | Qdrant `candidates` |
| F4 | **Role-specific interview generation** — questions targeted to the candidate's gaps + the role | "generates role-specific interviews" | Mastra agent |
| F5 | **Response evaluation + Validation Lab** — grade answers; timed, paste-blocked role-fit task | "evaluates responses" | Mastra + Enkrypt |
| F6 | **Recruiter-ready recommendation** — Shortlist / Needs-Lab / Reject + reason + top gap + 12-wk plan | "recruiter-ready hiring recommendations" | Mastra HITL |
| F7 | **Fairness + grounding + toxicity gate** (cross-cutting) — block ungrounded or biased output | "ensuring fairness and safety" | **Enkrypt** (hard gate) |
| F8 | **Cross-session memory** — learns the recruiter's bar from past decisions | "persistent memory" | Qdrant `outcomes` |

## 5. User journey

**Recruiter (primary loop):**
1. Paste a **JD / role brief** → agent extracts required skills + proof bar (F1).
2. Add candidates (upload resumes / point to GitHub handles / project URLs).
3. Agent builds each candidate's **proof skill graph** (F2) and **ranks** them (F3) — *evidence-grounded, bias-checked* before display (F7).
4. Recruiter reviews the ranked list: per candidate → match score, direct/transfer/missing proof, risk flags, top gap.
5. (Optional, deeper) Recruiter triggers a **role-specific interview** (F4); candidate completes it incl. a **Validation-Lab task**; agent **evaluates responses** (F5).
6. Agent surfaces a **safety-gated recommendation** (F6); workflow **suspends for the human** to decide **Shortlist / Needs-Lab / Reject-for-now**.
7. Decision is **persisted to memory** (F8) → future ranking warm-starts to this recruiter's bar.

**Candidate (self-serve, existing `/builder`):** submit GitHub/resume/projects → receive a proof profile + gap plan → improve and re-submit.

## 6. Expected outcomes

- **Faster first pass:** replace manual resume triage with a ranked, reasoned shortlist (target: minutes, not hours, per role).
- **Surfaced hidden talent:** measurable count of qualified **Tier-2/3 / non-pedigree** candidates ranked into the shortlist who keyword/pedigree filters would have dropped.
- **Zero ungrounded claims:** every skill shown is backed by a retrieved evidence vector; unsupported claims are demoted, never badged.
- **Auditable fairness:** recommendations invariant (within ε) to name/city/college proxies, with an Enkrypt audit trail per decision.
- **Recruiter-ready, not chatbot:** output is a decision + reason + next action, not a conversation.

## 7. Scope & non-goals

**In scope (R1 design → R2/finale build):** the 8 features above for a single role at a time, recruiter-facing.
**Non-goals (this hackathon):** ATS integrations, payments/billing, multi-tenant org management, automated outreach/messaging to candidates, fully autonomous hiring (a human **always** decides — by design).

## 8. Why now / novelty

- **TypeScript-first** end-to-end (HireGEN is already Next.js 16 / TS) — no Python gate.
- **Proof-not-pedigree + Tier-2/3 inclusion** — socially material in the Indian market.
- **Fairness as a hard gate, not a disclaimer** — Enkrypt blocks biased/ungrounded output *before* a recruiter sees it; the fairness golden-set doubles as a permanent regression suite. This is the differentiator, and it's defensible.

## 9. Official stack references used

- Mastra docs: https://mastra.ai/docs
- Mastra agents: https://mastra.ai/docs/agents/overview
- Mastra workflows: https://mastra.ai/docs/workflows/overview
- Qdrant docs: https://qdrant.tech/documentation/
- Enkrypt AI: https://www.enkryptai.com/
- Enkrypt Rayder: https://www.enkryptai.com/rayder
- Enkrypt Skill Sentinel: https://github.com/enkryptai/skill-sentinel

---

## 10. Requirement specification

This section is written as the evaluator-facing PRD layer. It converts the product narrative above into explicit, testable requirements with IDs, acceptance criteria, and traceability to the proposed Mastra/Qdrant/Enkrypt architecture.

### 10.1 Functional requirements

| ID | Requirement | Priority | Acceptance criteria | Traceability |
|---|---|---:|---|---|
| FR-HG-001 | The system shall ingest a free-text job description and extract role title, seniority, required skills, preferred skills, domain signals, and proof bar. | P0 | Given a JD, the output contains at least 8 normalized role requirements and separates must-have from preferred skills. | JD Analyzer Agent · Qdrant `role_intel` |
| FR-HG-002 | The system shall ingest candidate evidence from resume text, GitHub URLs, live project URLs, private-work notes, certifications, and interview responses. | P0 | Candidate evidence is accepted without requiring GitHub; missing GitHub must not create a penalty by itself. | Skill Graph Agent · Object Storage · Qdrant `evidence` |
| FR-HG-003 | The system shall create a candidate skill graph with evidence labels for each skill claim. | P0 | Every skill includes score, category, evidence source, evidence level, and confidence note. | Skill Graph Agent |
| FR-HG-004 | The system shall classify evidence as direct proof, logical transfer, candidate supplied, missing proof, or lab pending. | P0 | No recruiter-facing claim is shown without one of the five evidence labels. | Evidence model · Enkrypt grounding gate |
| FR-HG-005 | The system shall semantically rank candidates against the JD using role context and candidate evidence. | P0 | Ranked output includes fit score, score breakdown, matched proof, transfer evidence, missing proof, and next action. | Candidate Ranker Agent · Qdrant `candidates` |
| FR-HG-006 | The system shall generate role-specific interview questions based on the JD and candidate gaps. | P1 | At least 5 questions are generated per candidate and each question maps to a role requirement or identified gap. | Interview Generator Agent |
| FR-HG-007 | The system shall generate validation-lab tasks for uncertain or high-risk skills. | P1 | For every high-priority gap, the system can recommend a lab task with expected evidence and grading rubric. | Validation Lab workflow |
| FR-HG-008 | The system shall evaluate candidate responses against role-specific rubrics. | P1 | Evaluation output includes pass/risk flags, rubric score, evidence references, and human-review notes. | Response Evaluation Agent · Enkrypt |
| FR-HG-009 | The system shall produce recruiter-ready recommendations: Shortlist, Needs Lab, or Reject for Now. | P0 | Every recommendation includes why, key evidence, risk, and recommended next action. | Recruiter Copilot Agent · HITL workflow |
| FR-HG-010 | The system shall pause for human recruiter approval before any hiring decision is finalized. | P0 | No autonomous reject/shortlist action is executed without recruiter confirmation. | Mastra HITL suspend/resume |
| FR-HG-011 | The system shall remember recruiter decisions and outcomes for future role calibration. | P1 | Recruiter action and candidate outcome are stored as memory vectors and can be retrieved by similar roles. | Qdrant `outcomes` |
| FR-HG-012 | The system shall run safety, grounding, toxicity, and bias checks before displaying recommendations. | P0 | Unsafe, ungrounded, or biased outputs are blocked or marked for review before recruiter display. | Enkrypt safety gate |
| FR-HG-013 | The system shall expose an audit trail for score, evidence, generated questions, evaluations, and recruiter decisions. | P1 | A reviewer can inspect why a candidate received a score and which evidence influenced it. | Audit log · Qdrant · relational DB |
| FR-HG-014 | The system shall support candidate-facing proof readiness without charging candidates for core access. | P2 | Candidate can create/read a proof profile and roadmap without payment. | Candidate builder UI |

### 10.2 Non-functional requirements

| ID | Requirement | Target / threshold | Why it matters |
|---|---|---|---|
| NFR-HG-001 | Explainability | 100% of displayed skills and recommendations must include evidence labels. | Hiring decisions must be inspectable, not black-box. |
| NFR-HG-002 | Safety gate coverage | 100% of recruiter-facing AI outputs pass through Enkrypt or equivalent policy gate. | Hiring is high-stakes and bias-sensitive. |
| NFR-HG-003 | Latency | MVP target: first ranked shortlist under 60 seconds for 50 candidates; production target: async worker for larger batches. | Recruiter workflow should feel faster than resume screening. |
| NFR-HG-004 | Availability | MVP design supports graceful degradation if GitHub or LLM provider is unavailable. | Private/enterprise candidates should not be penalized due to external API failure. |
| NFR-HG-005 | Data minimization | Store only required candidate evidence, generated outputs, audit IDs, and recruiter decisions. | Reduces privacy risk and supports DPDP/GDPR posture. |
| NFR-HG-006 | Reproducibility | Same input evidence + same model/version configuration should produce traceable comparable outputs. | Prevents silent drift in recruiter decisions. |
| NFR-HG-007 | Observability | Every agent run should emit trace ID, prompt version, model, latency, token use, safety result, and output schema status. | Enables debugging and responsible AI audits. |
| NFR-HG-008 | Accessibility | Recruiter dashboard and candidate profile should meet WCAG AA contrast and keyboard navigation for core flows. | Hiring tools must be usable by diverse users. |
| NFR-HG-009 | Security | Secrets are server-only; uploaded resumes and evidence artifacts are private by default. | Candidate evidence can contain sensitive personal data. |

## 11. Compliance and responsible AI requirements

| ID | Requirement | Acceptance criteria |
|---|---|---|
| COMP-HG-001 | Consent capture | Candidate-facing upload flow must clearly state what data is processed and why before analysis begins. |
| COMP-HG-002 | Data retention | MVP profile retention defaults to 14 days unless the user explicitly saves or shares the profile. |
| COMP-HG-003 | Right to deletion | Production roadmap includes a deletion endpoint that removes profile records, artifact references, and vector memory by candidate/profile ID. |
| COMP-HG-004 | Bias-sensitive fields | Ranking must not use protected attributes such as gender, religion, caste, age, marital status, or disability. |
| COMP-HG-005 | Proxy monitoring | The system should monitor adverse impact across available non-sensitive proxy groups only for fairness auditing, not for ranking boosts. |
| COMP-HG-006 | 4/5ths rule check | Recruiter dashboards should surface a warning when selection-rate disparity crosses the 80% adverse-impact threshold. |
| COMP-HG-007 | Human accountability | The system is decision support only; final shortlist/reject decisions remain with the recruiter. |
| COMP-HG-008 | Grounding | Unsupported claims must be labeled as candidate-supplied or missing proof, not converted into verified skills. |

## 12. Agent prompt and output contracts

Each Mastra agent must use a constrained prompt with a typed output schema. The goal is not free-form chat; the goal is predictable hiring workflow outputs.

| Agent | Prompt objective | Required output schema |
|---|---|---|
| JD Analyzer Agent | Extract role requirements without adding imaginary skills. | `RoleRequirement[]`, seniority, must-have/preferred split, proof bar |
| Skill Graph Agent | Convert candidate evidence into normalized skills with source references. | `SkillGraph`, `EvidenceItem[]`, evidence levels |
| Candidate Ranker Agent | Compare role requirements against candidate proof. | `FitScore`, `score_breakdown`, matched/missing/transfer evidence |
| Interview Generator Agent | Create questions targeted to role gaps and uncertainty. | `InterviewQuestion[]` with linked requirement IDs |
| Response Evaluation Agent | Grade answers against rubric and expected evidence. | `EvaluationResult`, rubric scores, uncertainty notes |
| Recruiter Copilot Agent | Summarize candidate recommendation for human decision. | `Recommendation`, next action, risk flags, audit references |

Prompt style target: CRISPE-style instructions — Context, Role, Instruction, Steps, Parameters, Evaluation. Few-shot examples should include (a) strong direct proof, (b) logical transfer, (c) private enterprise work, (d) missing proof, and (e) biased or ungrounded output that must be blocked.

## 13. Traceability matrix

| Challenge requirement | HireGEN feature | PRD requirement IDs | Architecture component |
|---|---|---|---|
| Analyze job descriptions | JD understanding | FR-HG-001 | JD Analyzer Agent · Qdrant `role_intel` |
| Semantically rank candidates | Evidence-based ranking | FR-HG-003, FR-HG-004, FR-HG-005 | Candidate Ranker Agent · Qdrant |
| Generate role-specific interviews | Interview generation | FR-HG-006, FR-HG-007 | Interview Generator Agent · Validation Lab |
| Evaluate responses | Response evaluation | FR-HG-008 | Response Evaluation Agent · Enkrypt |
| Recruiter-ready recommendations | HITL recommendation | FR-HG-009, FR-HG-010 | Recruiter Copilot Agent · Mastra HITL |
| Ensure fairness and safety | Safety gate and audit | FR-HG-012, COMP-HG-004, COMP-HG-006 | Enkrypt safety gate |
| Persistent memory/RAG | Outcome memory | FR-HG-011 | Qdrant `outcomes` |
| Full-stack feasibility | Frontend/backend/data deployment | NFR-HG-003, NFR-HG-007, NFR-HG-009 | Next.js · Mastra · Qdrant · DB · storage |

## 14. MVP acceptance tests

1. **JD extraction test:** Given a Senior AI Systems Engineer JD, the system extracts AI/ML, Python, RAG, agentic workflows, cloud-native deployment, observability, and evaluation requirements.
2. **No GitHub bias test:** Given a strong enterprise/private-work profile without GitHub, the system must still produce a credible skill graph and mark GitHub as optional/missing, not as a disqualifier.
3. **Evidence label test:** Every recruiter-visible skill must show direct proof, logical transfer, candidate supplied, missing proof, or lab pending.
4. **Target-fit explanation test:** A low target-role score must explain which required skills are missing and how lab validation can close uncertainty.
5. **Interview generation test:** Generated questions must map to role requirements and candidate gaps.
6. **Safety gate test:** If the output uses unsupported demographic or pedigree assumptions, Enkrypt blocks the response.
7. **Human-in-loop test:** Recruiter recommendation cannot finalize until a recruiter selects Shortlist, Needs Lab, or Reject for Now.
8. **Outcome memory test:** A recruiter decision is stored and retrieved as context for a similar future role.

## 15. Observability and audit design

For Round-2 build, every workflow run should create a `trace_id` and record:

- agent name and version,
- prompt template version,
- input artifact IDs,
- model/provider,
- schema validation result,
- Enkrypt policy result,
- Qdrant collection/query IDs,
- latency and token usage,
- final recommendation,
- recruiter HITL decision,
- any override reason.

Recommended tools for implementation: OpenTelemetry-compatible traces, Langfuse or Arize Phoenix for LLM observability, and a lightweight audit table for recruiter decisions.

## 16. Build milestones if selected

| Date window | Milestone | Output |
|---|---|---|
| Jul 4–5 | Mastra workflow skeleton | TypeScript agents wired with typed schemas and mock data |
| Jul 6–7 | Qdrant memory + retrieval | Role intelligence, candidate evidence, and outcome collections |
| Jul 8 | Enkrypt gate | Safety, grounding, and bias checks before recruiter display |
| Jul 9 | Interview + response evaluation | Role-specific interview and validation-lab preview |
| Jul 10–11 | Recruiter dashboard polish | HITL actions, audit view, score explanation |
| Jul 12 | Finale demo | End-to-end JD → candidate ranking → interview → evaluation → recruiter recommendation |

---
*HireGEN · "Get hired on proof, not pedigree."*

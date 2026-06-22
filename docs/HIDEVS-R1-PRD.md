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
*HireGEN · "Get hired on proof, not pedigree."*

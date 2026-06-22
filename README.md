# HireGEN — HiDevs x Mastra Hackathon 2026

Round 1 submission for the **AI Hiring Copilot** track.

HireGEN is an agentic hiring copilot designed to help recruiters move beyond resume keywords and shortlist candidates using evidence-backed proof of capability.

## Submission Documents

- [Product Requirements Document](docs/HIDEVS-R1-PRD.md)
- [Full-Stack Architecture Document](docs/HIDEVS-R1-ARCHITECTURE.md)
- [Technical Architecture Diagram - PNG](assets/architecture/hiregen-hidevs-r1-technical-architecture.png)
- [Technical Architecture Diagram - PDF](assets/architecture/hiregen-hidevs-r1-technical-architecture.pdf)
- [Architecture Copilot Export - JSON](assets/architecture/hiregen-hidevs-r1-technical-architecture.json)

## Architecture Summary

HireGEN uses:

- **Mastra** for agent orchestration, workflow state, tool calls, and human-in-the-loop recruiter actions.
- **Qdrant** for role intelligence, candidate memory, evidence retrieval, and outcome memory.
- **Enkrypt AI** for grounding, bias/fairness, toxicity, PII, and interview-quality gates.
- **Next.js + TypeScript** for the recruiter-facing product surface and backend API layer.

Core workflow:

```text
ingest JD + candidates
-> build skill graph
-> retrieve role context
-> match and rank candidates
-> generate role-specific interview
-> evaluate candidate responses
-> run Enkrypt safety/eval gate
-> human recruiter decision
-> produce recommendation
-> persist outcome memory
```

## Round 1 Scope

Round 1 is a design submission: PRD plus full-stack architecture. The working Mastra/Qdrant/Enkrypt agent build is planned for the build sprint after Round 1 shortlisting.

# EPL Tutors — architecture case study

A production learning platform I designed, built, and operate for my own GCSE and A-level science students. The product repos stay private. This note is the public technical story.

**Live product:** [epltutors.com](https://epltutors.com) (tutoring site — not a portfolio page)  
**This write-up:** lives in the public [epltutors-architecture-showcase](https://github.com/Salama-Khan/epltutors-architecture-showcase) repo, not on the product domain  
**Student demo login:** available on request (synthetic account only — never a tutor login on the live database)

---

## The problem

Small-group tutoring produces a lot of signal that usually dies in a notebook: what was taught, what landed, what should be practised, and whether last month’s “they’ve got this” is still true. Parents want a weekly picture. The tutor needs a queue, not another spreadsheet.

I modelled that as a **syllabus knowledge graph** plus two separate write paths: homework compliance, and mastery.

## What a recruiter can click

1. Marketing site — the real tutoring product parents use. No employer chrome.
2. Student login (on request) — Knowledge Graph heatmap, one homework sprint if the vault has questions, weekly report / PDF.
3. This repo — schema and trade-offs. No source.

I do **not** share a tutor account on production. The operator APIs were built for a single tutor. A tutor session can see the full roster. Real students include minors. That is a hard no.

---

## Data model

```
Student ──< StudentAssessment >── Competency
   │                │                  │
   │                └── LessonLog      └── Prerequisite (edge)
   │
   ├── QuizAttempt          (objective % scores)
   ├── SprintAssignment     (homework; does not recolour the graph)
   └── QuestionBank         (tutor-approved MCQ vault)
```

**Competency** is one syllabus outcome (for example `1.2 Kinetic & GPE`) with exam-board tags (Combined / Triple / IGCSE, plus Edexcel- and Cambridge-only nodes). **Prerequisite** edges are the graph.

**StudentAssessment** is an append-only ledger. Latest row per `(student, competency)` is the mastery snapshot. `next_review_date` comes from a shared 1–5 → days table (1 day through 30 days). Older rows stay so a weekly digest can say improving vs slipping.

**Homework does not paint the map.** Sprint scores are compliance. Only Graph Practice (vault MCQs) writes mastery. That stops a lucky homework set from turning a topic green.

**Programme is an assignment, not a UI toggle.** GCSE Combined / Triple / exam board live on the student row so a shared laptop cannot flip another child’s map.

## AI, with a human in the loop

- Tutor pastes lesson notes → background job extracts topics + **evidence quotes** into the ledger (`gpt-4o-mini`, structured outputs).
- Draft MCQs go to a Dual-Track queue: assign as homework, send to the vault, or reject.
- The vault is the only source for Graph Practice. Unreviewed model output never colours mastery.

That is the product decision I would defend in an interview: generation is cheap; **publication is the control**.

## Stack I actually run

- API: FastAPI, SQLAlchemy 2, Pydantic v2, Alembic, JWT + bcrypt
- DB: PostgreSQL (Neon) — pooler for the API, direct host for migrations
- Frontend: Next.js on Vercel at epltutors.com
- API host: Docker on AWS EC2, Nginx, Let’s Encrypt (`api.epltutors.com`)
- Tests: 100+ pytest cases on in-memory SQLite (product flows, boards, reports)

I am not selling this as a multi-tenant SaaS. It is a single-operator tool with a real domain model. Tenant isolation (tutor-scoped roster and interventions) is the next gate if it becomes a product for other tutors.

## Why this exists next to my data work

The interesting part is not the React shell. It is the graph, the ledger, the board maps, and the review loop. Public repos such as the architecture showcase and the Mistral fine-tune pipeline are the complementary “data / modelling” artefacts. This platform is where that model has to survive contact with real lessons.

## What I would tell an interviewer

- Why homework and mastery are different writers.
- Why assessments are append-only and ordered by `created_at`, not `next_review_date`.
- Why a tutor demo on the live database is unsafe, and how I would add `tutor_id` filters before selling this to other centres.

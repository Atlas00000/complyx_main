# Comprehensive Weekly Implementation: Inline Assessment in Chat

**Combines:** assessment-mode.md + framework.md  
**Aligned with:** Current codebase structure, component/modularity approach (one file at a time)

---

## Table of Contents

1. [Overview & Summary](#overview--summary)
2. [Framework & Scoring Reference](#framework--scoring-reference)
3. [Codebase Alignment](#codebase-alignment)
4. [Weekly Implementation Phases](#weekly-implementation-phases)
5. [File-by-File Implementation Map](#file-by-file-implementation-map)
6. [Dependencies & Checklist](#dependencies--checklist)

---

## Overview & Summary

### Goal

Enable users to take IFRS S1/S2 readiness assessments **inside the chat** without navigating to another screen. Assessment type is selectable (Quick, Micro, Full). Chat uses assessment data for personalized responses.

### Phases at a Glance

| Phase | Weeks | Focus |
|-------|-------|--------|
| **1** | 1–2 | Backend: schema, APIs, question bank, scoring |
| **2** | 3–4 | Chat: type selection, text-based assessment flow |
| **3** | 5–6 | Rich blocks (MC, Yes/No, Scale), scoring integration |
| **4** | 7–8 | Assessment context in chat, micro-assessments, contextual questions |

### Component/Modularity Approach

- **Work on ONE file at a time** to avoid timeouts and maintain quality
- Each component/service is self-contained and testable
- Clear file paths for every deliverable

---

## Framework & Scoring Reference

### Assessment Types

| Type | Questions | Duration | Question IDs |
|------|-----------|----------|--------------|
| **Quick** | 5–7 | ~5 min | G1, G2, S1, R1, M1 (+ S3, M3 optional) |
| **Micro** | 5 per topic | ~3 min | Governance: G1–G5; Strategy: S1–S5; Risk: R1–R5; Metrics: M1–M5 |
| **Full** | 20 | ~15 min | G1–G5, S1–S5, R1–R5, M1–M5 |

### Pillars

1. **Governance** – Oversight, roles, processes  
2. **Strategy** – Approach, integration, targets  
3. **Risk** – Identification, assessment, management  
4. **Metrics & Targets** – KPIs, measurement, disclosure  

### Question Types & Scoring

| Type | % | Scoring |
|-----|---|---------|
| Multiple choice | 50% | 0/25/50/75/100 by option |
| Yes/No | 25% | Yes=100, No=0 |
| Scale (1–5) | 15% | 1→0%, 2→25%, 3→50%, 4→75%, 5→100% |
| Open text | 10% | Context only (no score) |

### Scoring Logic

- **Category score** = average of question scores in that pillar  
- **Overall score** = weighted average of pillars (25% each)  
- **Gap** = question or pillar < 50%  
- **Readiness bands:** 0–19%, 20–39%, 40–59%, 60–79%, 80–100%  

### Selectable Flow

```
User triggers assessment
    → AI: "Which type? Quick, Micro, or Full?"
    → User selects (buttons or text)
    → If Micro: "Which topic?" → Governance/Strategy/Risk/Metrics
    → AI starts appropriate question set
```

*Full question bank: see `framework.md` Part 1.5*

---

## Codebase Alignment

### Existing Structure (from README / roadmap)

```
ifrsbot/
├── client/
│   ├── app/                    # Next.js App Router
│   │   ├── dashboard/
│   │   ├── auth/
│   │   └── admin/
│   ├── components/
│   │   ├── chat/               # ChatInterface, MessageBubble, ChatInput, etc.
│   │   ├── assessment/         # ProgressTracker, ComplianceMatrix, ReadinessScore
│   │   ├── dashboard/
│   │   └── ui/
│   ├── lib/
│   ├── hooks/
│   ├── stores/                 # chatStore, assessmentStore
│   └── types/
│
├── server/
│   ├── src/
│   │   ├── routes/             # chat, assessment, auth, etc.
│   │   ├── controllers/
│   │   ├── services/
│   │   │   ├── ai/
│   │   │   ├── assessment/
│   │   │   ├── auth/
│   │   │   └── knowledge/
│   │   ├── middleware/
│   │   └── utils/
│   └── prisma/
│       ├── schema.prisma
│       ├── migrations/
│       └── seeds/
```

### Existing Prisma Models (Relevant)

| Model | Status | Notes |
|-------|--------|-------|
| `User` | ✅ Exists | Links to Assessment |
| `Session` | ✅ Exists | Links to Assessment |
| `Assessment` | ✅ Exists | Add `assessmentType`, `microTopic` |
| `QuestionCategory` | ✅ Exists | governance, strategy, risk, metrics |
| `Question` | ✅ Exists | Add `questionSet` (quick/micro_governance/full) |
| `Answer` | ✅ Exists | — |
| `Score` | ✅ Exists | — |

### Schema Additions Required

- `Assessment.assessmentType` – `quick` | `micro` | `full`
- `Assessment.microTopic` – `governance` | `strategy` | `risk` | `metrics` (nullable)
- `Question.questionSet` – JSON or enum for which assessment types include this question

---

## Weekly Implementation Phases

---

## Phase 1: Foundation (Weeks 1–2)

### Week 1: Data Model & Question Bank

| Day | Task | File(s) |
|-----|------|---------|
| 1 | Prisma migration: add `assessmentType`, `microTopic` to Assessment; add `questionSet` to Question | `server/prisma/migrations/` |
| 2 | API: `POST /api/assessment/start`, `POST /api/assessment/answer`, `GET /api/assessment/status` | `server/src/routes/assessment.ts`, `server/src/controllers/assessmentController.ts` |
| 3 | Seed question bank (20 questions from framework.md) with pillar, type, options, scores, questionSet | `server/prisma/seeds/assessmentQuestionSeed.ts` |
| 4 | Assessment service: start session, submit answer, get next question | `server/src/services/assessment/inChatAssessmentService.ts` |
| 5 | Scoring logic: category scores, overall score, gap detection | `server/src/services/assessment/scoringService.ts` |

**Deliverables:** Schema, APIs, question bank, scoring service.

---

### Week 2: Assessment Type Selection & Basic Flow

| Day | Task | File(s) |
|-----|------|---------|
| 1 | Define question subsets: Quick (5–7), Micro (5 per topic), Full (20) | `server/src/services/assessment/questionSetService.ts` |
| 2 | API: accept `type` and `topic` on start; return correct question set | `server/src/controllers/assessmentController.ts` |
| 3 | Chat: detect "start assessment" intent; enter assessment mode | `client/stores/chatStore.ts` or `client/stores/assessmentStore.ts` |
| 4 | Chat: AI asks for type; render Quick/Micro/Full buttons | `client/components/chat/AssessmentTypeSelector.tsx` |
| 5 | Chat: if Micro, ask topic; render Governance/Strategy/Risk/Metrics buttons | `client/components/chat/AssessmentTopicSelector.tsx` |

**Deliverables:** Type selection in chat, correct question set per type.

---

## Phase 2: Chat Integration (Weeks 3–4)

### Week 3: Text-Based Assessment Flow

| Day | Task | File(s) |
|-----|------|---------|
| 1 | System prompt for assessment mode (type-aware) | `server/src/services/ai/prompts/assessmentPrompt.ts` |
| 2 | AI sends Q1 → user replies → store answer → AI sends Q2 | `server/src/controllers/chatController.ts`, `server/src/services/assessment/inChatAssessmentService.ts` |
| 3 | Progress: "Question 3 of 8" (Quick) / "Question 2 of 5" (Micro) / "Question 10 of 18" (Full) | `client/components/chat/AssessmentProgressIndicator.tsx` |
| 4 | Completion: compute scores, gaps, readiness band; AI sends summary | `server/src/services/assessment/scoringService.ts`, `server/src/services/ai/prompts/assessmentPrompt.ts` |
| 5 | Exit assessment mode; persist results | `client/stores/assessmentStore.ts`, `server/src/services/assessment/inChatAssessmentService.ts` |

**Deliverables:** End-to-end text-based assessment for all types.

---

### Week 4: Polish & Edge Cases

| Day | Task | File(s) |
|-----|------|---------|
| 1 | Skip / "I don't know" handling | `server/src/services/assessment/inChatAssessmentService.ts`, `client/components/chat/ChatInput.tsx` |
| 2 | Mid-assessment type switch (confirm, save, restart) | `client/components/chat/AssessmentTypeSwitchConfirm.tsx` |
| 3 | Resume partial assessment | `server/src/services/assessment/inChatAssessmentService.ts`, `client/hooks/useAssessment.ts` |
| 4 | Progress indicator in chat UI | `client/components/chat/AssessmentProgressIndicator.tsx` |
| 5 | Suggested prompts: "Start assessment", "Quick check", etc. | `client/data/promptTemplates.ts` or `client/data/suggestedPrompts.ts` |

**Deliverables:** Robust flow, UX polish.

---

## Phase 3: Rich Blocks & Scoring (Weeks 5–6)

### Week 5: Rich Message Blocks

| Day | Task | File(s) |
|-----|------|---------|
| 1 | Message schema: `type: 'assessment_question'`, payload for MC/YesNo/Scale | `client/types/chat.ts`, `server/src/types/assessment.ts` |
| 2 | Server: return structured question blocks for applicable questions | `server/src/services/assessment/inChatAssessmentService.ts` |
| 3 | Client: `AssessmentQuestionBubble` (MC, Yes/No, Scale) | `client/components/chat/AssessmentQuestionBubble.tsx` |
| 4 | Client: submit selection → store answer → next question | `client/components/chat/AssessmentQuestionBubble.tsx`, `client/hooks/useAssessment.ts` |
| 5 | Mixed flow: text questions + rich blocks where applicable | `client/components/chat/MessageBubbleContent.tsx` |

**Deliverables:** Interactive question blocks in chat.

---

### Week 6: Scoring Integration & Gaps

| Day | Task | File(s) |
|-----|------|---------|
| 1 | Map MC options to 0/25/50/75/100 | `server/src/services/assessment/scoringService.ts` |
| 2 | Map Scale 1–5 to 0–100% | `server/src/services/assessment/scoringService.ts` |
| 3 | Category scores and overall weighted score | `server/src/services/assessment/scoringService.ts` |
| 4 | Gap identification (questions < 50%) | `server/src/services/assessment/scoringService.ts` |
| 5 | Readiness band and summary message in completion | `server/src/services/ai/prompts/assessmentPrompt.ts` |

**Deliverables:** Full scoring and gap reporting.

---

## Phase 4: Context & Advanced (Weeks 7–8)

### Week 7: Assessment Context in Chat

| Day | Task | File(s) |
|-----|------|---------|
| 1 | Assessment summary service (scores, gaps, key answers) | `server/src/services/assessment/assessmentSummaryService.ts` |
| 2 | Inject summary into chat context/system prompt | `server/src/services/ai/utils/contextBuilder.ts`, `server/src/controllers/chatController.ts` |
| 3 | AI uses assessment data for personalized answers | `server/src/services/ai/prompts/chatPrompt.ts` or equivalent |
| 4 | Handle no assessment / partial assessment | `server/src/services/assessment/assessmentSummaryService.ts` |
| 5 | UI hint: "Using your assessment to personalize responses" | `client/components/chat/AssessmentContextHint.tsx` |

**Deliverables:** Chat aware of assessment results.

---

### Week 8: Contextual Questions & Wrap-Up

| Day | Task | File(s) |
|-----|------|---------|
| 1 | Detect assessment-relevant info in normal chat | `server/src/services/assessment/contextualAnswerService.ts` |
| 2 | Map to framework; store as contextual answers | `server/src/services/assessment/contextualAnswerService.ts` |
| 3 | Update scores when new info is captured | `server/src/services/assessment/scoringService.ts` |
| 4 | Tune when to ask vs. answer | `server/src/services/ai/prompts/chatPrompt.ts` |
| 5 | Testing, docs, final polish | — |

**Deliverables:** Contextual data capture and integration.

---

## File-by-File Implementation Map

### Server (One File at a Time)

| File | Phase | Purpose |
|------|-------|---------|
| `prisma/migrations/xxx_add_assessment_type/migration.sql` | W1 | Schema changes |
| `prisma/seeds/assessmentQuestionSeed.ts` | W1 | Question bank |
| `src/services/assessment/inChatAssessmentService.ts` | W1, W2, W3 | Core assessment flow |
| `src/services/assessment/scoringService.ts` | W1, W6 | Scoring logic |
| `src/services/assessment/questionSetService.ts` | W2 | Question subsets |
| `src/controllers/assessmentController.ts` | W1, W2 | Assessment API |
| `src/routes/assessment.ts` | W1 | Route wiring |
| `src/services/ai/prompts/assessmentPrompt.ts` | W3, W6 | Assessment prompts |
| `src/services/assessment/assessmentSummaryService.ts` | W7 | Summary for chat context |
| `src/services/assessment/contextualAnswerService.ts` | W8 | Contextual answers (detect pillar, map to question, store) |
| `src/services/ai/prompts/chatPrompt.ts` | W8 | When to use assessment-relevant info (ask vs. answer) |
| `src/services/ai/utils/contextBuilder.ts` | W7, W8 | Inject assessment + contextual prompt into context |
| `server/scripts/testWeek8Contextual.ts` | W8 | Test contextual detection and storage |

### Client (One File at a Time)

| File | Phase | Purpose |
|------|-------|---------|
| `stores/assessmentStore.ts` | W2, W3 | Assessment state |
| `components/chat/AssessmentTypeSelector.tsx` | W2 | Quick/Micro/Full buttons |
| `components/chat/AssessmentTopicSelector.tsx` | W2 | Governance/Strategy/Risk/Metrics |
| `components/chat/AssessmentProgressIndicator.tsx` | W3, W4 | Progress UI |
| `components/chat/AssessmentQuestionBubble.tsx` | W5 | MC, Yes/No, Scale blocks |
| `components/chat/AssessmentTypeSwitchConfirm.tsx` | W4 | Mid-assessment switch |
| `components/chat/AssessmentContextHint.tsx` | W7 | "Using your assessment" hint |
| `hooks/useAssessment.ts` | W3, W4, W5 | Assessment hook |
| `types/chat.ts` | W5 | Message types |
| `data/suggestedPrompts.ts` or `promptTemplates.ts` | W4 | Assessment prompts |

### Integration Points (Existing Files to Modify)

| File | Changes |
|------|---------|
| `server/src/controllers/chatController.ts` | Assessment mode detection, context injection |
| `client/components/chat/ChatInterface.tsx` | Render assessment components, progress |
| `client/components/chat/MessageBubbleContent.tsx` | Render `AssessmentQuestionBubble` for rich blocks |

---

## Dependencies & Checklist

### Dependency Graph

```
Week 1 (Schema, APIs, Question Bank, Scoring)
    → Week 2 (Type Selection, Question Sets)
    → Week 3 (Chat Flow)
    → Week 4 (Polish)
    → Week 5 (Rich Blocks)
    → Week 6 (Scoring Integration)
    → Week 7 (Context)
    → Week 8 (Contextual + Wrap-up)
```

### Summary Timeline

| Week | Focus |
|------|--------|
| 1 | Schema, APIs, question bank, scoring |
| 2 | Type selection (Quick/Micro/Full), topic for Micro |
| 3 | Text-based assessment flow for all types |
| 4 | Edge cases, skip, resume, progress UI |
| 5 | Rich blocks (MC, Yes/No, Scale) |
| 6 | Scoring integration, gaps, readiness bands |
| 7 | Assessment context in chat |
| 8 | Contextual questions, testing, polish |

### Implementation Guidelines

1. **One file at a time** – Complete and test each file before moving on  
2. **Modular design** – Each component/service is self-contained  
3. **Clear interfaces** – TypeScript types for props and data  
4. **Incremental integration** – Integrate components one at a time  
5. **Design system** – Follow `client/docs/UI_PHILOSOPHY.md` for styling  

---

## Related Documents

- **framework.md** – Full question bank, scoring details, question mapping  
- **assessment-mode.md** – Original assessment mode brainstorm  
- **client/docs/UI_PHILOSOPHY.md** – Design system and component guidelines  
- **roadmap.md** – Overall project roadmap and component structure  

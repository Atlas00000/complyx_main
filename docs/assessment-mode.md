# Practical Path: Inline Assessment in Chat

## Overview

| Phase | Focus | Duration |
|-------|--------|----------|
| **Phase 1** | Assessment mode (text-based) | Weeks 1–3 |
| **Phase 2** | Rich message blocks | Weeks 4–5 |
| **Phase 3** | Assessment context in chat | Week 6 |
| **Phase 4** | Micro-assessments + contextual questions | Weeks 7–8 |

---

## Phase 1: Assessment Mode (Weeks 1–3)

### Week 1: Backend & Data Model

**Goals:** Define assessment flow and persistence.

- **Day 1–2: Schema & API**
  - Add `ChatMessage` (or equivalent) if chat history is persisted.
  - Add `AssessmentSession` (or extend `Assessment`) for in-chat assessments.
  - Link assessment to user/session.
  - API: `POST /api/assessment/start`, `POST /api/assessment/answer`, `GET /api/assessment/status`.

- **Day 3: Question flow**
  - Define question sequence (or load from existing question bank).
  - Implement "get next question" logic.
  - Store answers and compute basic scores.

- **Day 4–5: Assessment service**
  - Service to start assessment, submit answers, compute progress.
  - Simple scoring (e.g. per category, overall %).
  - End-of-assessment summary (scores, gaps).

---

### Week 2: Chat Integration – Assessment Mode

**Goals:** Start and run assessment inside the chat.

- **Day 1: Mode detection**
  - Detect "start assessment" intent (e.g. suggested prompt, keywords).
  - Set `assessmentMode: true` in chat state.
  - Persist mode in session/context.

- **Day 2: AI flow**
  - System prompt for assessment mode: "You are conducting an IFRS readiness assessment. Ask one question at a time…"
  - Map AI questions to question bank (or generate from framework).
  - Parse user replies as answers and store them.

- **Day 3: Question–answer loop**
  - AI sends Q1 → user replies → store answer → AI sends Q2.
  - Handle "skip" or "I don't know".
  - Track progress (e.g. "Question 3 of 12").

- **Day 4: Completion**
  - Detect when all questions are done.
  - Compute scores and gaps.
  - AI sends summary in chat.
  - Exit assessment mode.

- **Day 5: Polish**
  - Progress indicator in chat UI.
  - Suggested prompt: "Start IFRS readiness assessment".
  - Basic error handling and edge cases.

---

### Week 3: Testing & Refinement

**Goals:** Stabilize Phase 1 and fix issues.

- **Day 1–2: End-to-end testing**
  - Start assessment → answer all → receive summary.
  - Test skip, partial completion, resume.
  - Verify scores and stored answers.

- **Day 3: Edge cases**
  - User switches topic mid-assessment.
  - Long or unclear answers.
  - Session refresh / reconnect.

- **Day 4: UX**
  - Clear "assessment in progress" state.
  - Smooth transition from assessment to normal chat.

- **Day 5: Documentation**
  - Update docs for assessment flow.
  - Note any schema/API changes.

---

## Phase 2: Rich Message Blocks (Weeks 4–5)

### Week 4: Message Types & Rendering

**Goals:** Support structured question blocks in chat.

- **Day 1: Message schema**
  - Extend message model: `type: 'text' | 'assessment_question'`.
  - Payload: `questionId`, `questionType`, `options`, `scale`, etc.

- **Day 2: Server**
  - API returns structured question blocks.
  - Support multiple choice, yes/no, scale (1–5).
  - Store selected option as answer.

- **Day 3: Client components**
  - `AssessmentQuestionBubble` for multiple choice.
  - `ScaleQuestionBubble` for 1–5.
  - `YesNoQuestionBubble`.
  - Render inside chat message list.

- **Day 4: Integration**
  - In assessment mode, send rich blocks instead of plain text where applicable.
  - Handle submit/selection and send answer to backend.
  - Show next question (text or rich).

- **Day 5: Mixed flow**
  - Support mix of text questions and rich blocks.
  - Fallback for text-only clients if needed.

---

### Week 5: Polish & Migration

**Goals:** Make rich blocks the default for structured questions.

- **Day 1–2: Styling**
  - Match design system.
  - Hover, focus, selected states.
  - Mobile layout.

- **Day 3: Migration**
  - Convert existing structured questions to rich blocks.
  - Keep text fallback for open-ended questions.

- **Day 4–5: Testing**
  - All question types.
  - Accessibility (keyboard, screen reader).
  - Performance with many options.

---

## Phase 3: Assessment Context in Chat (Week 6)

### Week 6: Chat Uses Assessment Data

**Goals:** Inject assessment summary into chat context.

- **Day 1: Assessment summary service**
  - Function to build summary from latest assessment:
    - Overall score, category scores, top gaps.
  - Optional: short summary of key answers.
  - Handle "no assessment" and "in progress".

- **Day 2: Context injection**
  - In chat API, load assessment summary for user.
  - Add to system prompt or context:
    - "User assessment: [summary]. Use this to personalize answers."
  - Ensure it's included for both RAG and non-RAG flows.

- **Day 3: Prompt tuning**
  - Instruct AI to use assessment data when relevant.
  - Avoid over-referencing when not needed.
  - Test with and without assessment.

- **Day 4: Edge cases**
  - User with no assessment.
  - Partial assessment.
  - Multiple assessments (use latest).

- **Day 5: Transparency**
  - Add UI hint: "Using your assessment to personalize responses."
  - Optional: "Don't use my assessment" toggle for later.

---

## Phase 4: Micro-Assessments & Contextual Questions (Weeks 7–8)

### Week 7: Micro-Assessments

**Goals:** Short, topic-specific assessments in chat.

- **Day 1: Micro-assessment definitions**
  - Define 3–5 micro-assessments (e.g. governance, strategy, metrics).
  - Each: 3–5 questions, clear scope.
  - Map to question bank or create new ones.

- **Day 2: Triggering**
  - Suggested prompts: "Quick governance check", "Quick metrics check".
  - Or AI offers: "Want a quick 3-question governance check?"
  - Reuse assessment mode flow with shorter question set.

- **Day 3: Flow**
  - Start micro-assessment → ask 3–5 questions → immediate feedback.
  - Store results and link to user's assessment profile.
  - Aggregate into overall readiness over time.

- **Day 4: UI**
  - "Quick check" prompts.
  - Short progress indicator (e.g. "2 of 3").
  - Inline result: "Governance: ~70% ready."

- **Day 5: Testing**
  - Each micro-assessment.
  - Integration with main assessment and chat context.

---

### Week 8: Contextual Questions

**Goals:** Use normal chat to collect assessment-relevant answers.

- **Day 1: Intent detection**
  - Identify when user shares assessment-relevant info (governance, strategy, metrics, etc.).
  - Extract structured data from free text (e.g. "we have board oversight").

- **Day 2: Mapping to framework**
  - Map extracted info to assessment questions/requirements.
  - Store as "contextual answers" linked to assessment.
  - Update scores when new info is captured.

- **Day 3: AI behavior**
  - Update system prompt: "When the user shares relevant info, treat it as assessment data."
  - Optional: ask follow-up questions to clarify.
  - Avoid turning every message into an assessment question.

- **Day 4: Balance**
  - Tune when to ask vs. when to just answer.
  - Avoid feeling like an interrogation.
  - Test with real conversation flows.

- **Day 5: Integration & wrap-up**
  - Ensure contextual answers feed into assessment summary.
  - Verify chat context uses updated assessment data.
  - Final testing and documentation.

---

## Summary Timeline

| Week | Focus |
|------|--------|
| 1 | Backend: schema, API, question flow, scoring |
| 2 | Chat: assessment mode, AI flow, completion |
| 3 | Testing, edge cases, UX polish |
| 4 | Rich blocks: schema, server, client components |
| 5 | Rich blocks: styling, migration, testing |
| 6 | Assessment context in chat |
| 7 | Micro-assessments |
| 8 | Contextual questions, integration, wrap-up |

---

## Dependencies

- Week 2 depends on Week 1 (backend).
- Week 4–5 depend on Phase 1 (assessment mode).
- Week 6 depends on Phase 1 (assessment data).
- Week 7–8 depend on Phase 1 and 3 (assessment flow + context).

Phases can be adjusted (e.g. 2 weeks for Phase 1) depending on team size and complexity of the framework.

# TenMinute Coach 🧠⏱️
**An agentic 10-minute daily skill coach that adapts your 4-week plan based on what you actually do.**
Built for hackathons where *functionality + real-world relevance + LLM/agent design + evaluation/observability* matter.

TenMinute Coach is an AI agent that turns any learning goal into a 4-week micro-plan, delivers a daily 10-minute session, checks understanding (quiz/reflection), and adapts the plan based on performance and missed days—fully instrumented with Opik traces, evals, and dashboards.

---

## Why this exists
Most people don’t fail goals because they lack motivation—they fail because:
- plans are unrealistic,
- daily effort is inconsistent,
- feedback loops are missing,
- and tools don’t adapt when life happens.

TenMinute Coach makes progress *frictionless*: **10 minutes/day**, personalized, measurable, and continuously adjusted.

---

## What it does
### Core workflow
1. **Goal Intake**
   - User chooses a goal (e.g., “learn SQL basics”, “interview prep”, “journaling consistency”)
   - Optional: timeframe, level, constraints (busy days, preferred time), learning style

2. **4-Week Micro-Plan Generator**
   - Produces a structured plan: Week → Day → objective → session type
   - Includes review days + checkpoint days automatically

3. **Daily 10-Minute Session**
   - Generates a tight session (reading + example + short exercise)
   - “One screen” style: no long lectures

4. **Quick Check**
   - **Learning goals**: 3–5 question quiz (auto-graded)
   - **Reflection goals**: short prompts + rubric scoring (consistency, specificity, insight)

5. **Adaptive Re-planning**
   - Adjusts next sessions based on:
     - quiz score / rubric score
     - streak + missed days
     - difficulty feedback (“too easy / just right / too hard”)
   - Never punishes: it compresses and recovers gracefully

---

## Demo-ready user stories
- “I want to learn SQL in January” → gets a 4-week plan + day 1 lesson + quiz → plan adapts after quiz.
- Miss 3 days → coach returns with a “re-entry” session + plan compression.
- “I’m tired today” → switches to lighter session type (review, flashcards, recap).

---

## Agent design (how it works)
### Agents / modules
- **Planner Agent**
  - Creates the initial 4-week micro-plan from user goal + constraints
- **Session Agent**
  - Produces the daily 10-minute content
- **Checker Agent**
  - Builds the quiz/reflection check + grading rubric
- **Coach Agent**
  - Interprets performance + updates plan (next 1–3 days + weekly objectives)

### Memory
- Lightweight, explicit memory (no mystery):
  - user goal + preferences
  - current plan (structured JSON)
  - progress history (scores, misses, feedback)
  - “struggle tags” (e.g., joins, anxiety, consistency)

### Tooling
- Retrieval (optional but powerful):
  - curated sources for factual topics (e.g., SQL reference notes)
  - local knowledge base for each skill track

---

## Observability & evaluation (Opik-first)
This project is built to *win on evaluation and observability*.

### Tracing (every daily run)
Each session creates a trace with spans:
- `goal_intake`
- `plan_generation`
- `daily_session_generation`
- `quiz_generation`
- `grading`
- `coaching_update`
- `memory_write`

Captured metadata:
- goal type, day index, difficulty, token usage, latency
- quiz score, confidence, user feedback
- plan changes (diff)

### Evals (automated + human)
We ship an `evals/` folder with datasets and scorers:

#### 1) Plan Quality Eval
**Input**: goal + constraints  
**Output**: plan JSON  
**Checks**:
- feasibility (10 min/day, realistic pacing)
- coverage (progression, review, checkpoints)
- clarity (each day has objective + activity)
- personalization (uses constraints)

#### 2) Session Factuality / Groundedness Eval (for factual tracks like SQL)
- detect unsupported claims
- verify examples vs reference snippets (when retrieval enabled)

#### 3) Coaching Helpfulness Eval
- does feedback reference user performance?
- does it propose a concrete next step?
- does it maintain supportive tone?

#### 4) Safety / Tone Eval
- no shaming language
- avoids overreach (especially for journaling/mental wellness adjacent prompts)
- toxicity & bias checks

### Dashboard (tiny but powerful)
A simple dashboard page shows:
- active users, streaks, completion rate
- plan adherence uplift (baseline vs adapted)
- average quiz score over time
- failure cases with trace links

---

## MVP scope (hackathon-friendly)
### Must-have (MVP)
- Goal intake UI
- Plan generator (4 weeks)
- Daily session generator (10 minutes)
- Quiz/reflection check + scoring
- Adaptive next-day update
- Opik tracing for the entire pipeline
- Minimal dashboard

### Nice-to-have
- Calendar integration (suggest time)
- “Energy level” toggle
- Track templates (SQL, interviews, journaling, language)
- Retrieval with curated docs
- Export plan to Markdown/Notion

---

## Tech stack (suggested)
You can implement this in either Python or TypeScript. One clean option:

### Frontend
- Next.js + Tailwind
- Simple auth (Clerk / NextAuth)

### Backend
- FastAPI (Python) **or** Next.js API routes (TS)
- Postgres (Supabase) for plans + progress
- Redis (optional) for caching

### LLM / Agents
- OpenAI / Gemini / Claude (pluggable)
- Structured outputs (JSON schema) for plans/sessions

### Observability
- **Opik** for traces, eval runs, datasets, dashboards

---

## Data model (minimal)
### `User`
- id, name, timezone

### `Goal`
- id, user_id
- title, category, level, constraints_json

### `Plan`
- id, goal_id
- plan_json (weeks/days)
- created_at, updated_at

### `SessionRun`
- id, plan_id, day_index
- session_content_md
- check_payload_json
- score, feedback
- trace_id (Opik link key)
- created_at

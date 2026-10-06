# The Renewals Desk

**Live product:** [https://www.therenewalsdesk.com](https://www.therenewalsdesk.com)

A focused web chatbot that helps marketing operations and martech leaders decide what to do about **one** SaaS renewal at a time. It turns scattered ownership, usage, cost, dependency, and contract facts into a short, decision-ready assessment with a clear next step.

This repository is a **portfolio showcase** (product narrative and architecture). Application source code is private.

![The Renewals Desk opening screen: the wordmark, the opening question, the reply box, and a footer naming the builder and the OpenAI models in use](docs/landing.png)

One question to start. The desk asks for the company website and the tool under review. Public research on the company and the vendor runs in the background at set points in the review. Those findings go into the evidence record. They are not pasted into the chat.

![An intake conversation: the desk asks what the tool is used for and what prompted the review, then asks about cost, contract end date, auto-renewal and notice, and who holds approval authority](docs/conversation.png)

Intake asks only what could change the recommendation, and follows up on the answers rather than working through a fixed form. Commercial arithmetic is done in application code, not by the model. In this example a quoted 22% increase becomes the renewal figure. The company is fictional.

---

## Problem

Renewal decisions are hard when evidence is incomplete and spread across Marketing Ops, IT, Finance, Procurement, and Legal. Feature overlap alone does not prove a tool can be removed safely. Leaders need a timely recommendation that is honest about confidence, tradeoffs, and what still needs to be confirmed.

## What it does

1. **Intake conversation.** Asks for the company (or website) and the tool under review, then gathers purpose, ownership, commercial terms, usage, dependencies, overlap candidates, and vendor context through consequential follow-ups (typed or pasted text only; no file uploads).
2. **Structured evidence.** Maintains an in-session evidence record with value, source, and state (answered, unknown, conflicting, not yet asked). Does not invent missing contract terms.
3. **Public research.** At defined milestones, researches the company and the vendor/product into the evidence record (with sources), without dumping research into the chat.
4. **Deterministic deadline math.** Calculates notice deadlines and runway in application code (not via the language model), and surfaces timing when it materially matters.
5. **Formal assessment.** A separate assessment pass produces a compact write-up:
   - Immediate contract action (renew, renegotiate, extend, defer, or do not renew)
   - Longer-term direction (retain, sunset, consolidate, or investigate further)
   - Reasoning and evidence (supported, provisional, or insufficient information)
   - Action plan with owners and next steps
6. **Revision.** New evidence updates the recommendation; older assessments are superseded in the thread.

Sessions are **not saved** after the tab closes. Organizational approval stays with the user.

---

## Architecture (high level)

```text
┌─────────────────────────────────────────────────────────┐
│  Browser (Next.js App Router UI)                        │
│  • Single-column chat shell                             │
│  • In-memory session evidence + messages                │
│  • Streams assistant replies and assessments            │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│  API layer                                              │
│  • Chat          → streaming intake replies             │
│  • Extract       → structured evidence updates          │
│  • Research      → milestone company / vendor research  │
│  • Assess        → streaming formal assessment          │
│  • Bot-focused rate limits on public API routes         │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│  Domain logic                                           │
│  • Evidence merge and conflict handling                 │
│  • Commercial-term and notice-deadline calculation      │
│  • Assessment readiness / handoff                       │
│  • Role-separated model calls (chat, extract,           │
│    research, assessment) via configuration              │
└─────────────────────────────────────────────────────────┘
```

**Design choices that matter for the product:**

| Choice | Why |
|--------|-----|
| Intake and assessment are separate passes | Questions stay investigative; the formal write-up stays compact and consistent |
| Deadline arithmetic is deterministic | Avoids LLM date errors; runway uses the user's local calendar date |
| Research lands in evidence, not chat dumps | Keeps the conversation readable while still grounding recommendations |
| Session-only state | Fits a focused review tool without accounts or stored customer data |
| Scoped to one tool per review | Prevents whole-stack sprawl; overlap is discussed only as it affects the renewal |

---

## Features

- Focused renewal chatbot for B2B marketing technology decisions
- Warm, desk-inspired UI (mahogany / ivory) with clear brand hierarchy
- Streaming responses for chat and assessment
- Markdown rendering for readable assistant replies
- In-session structured evidence across intake categories
- Milestone-based company and vendor web research with sources
- Application-calculated notice deadline and runway when timing is known
- Four-section assessment with copy-to-clipboard
- Clear recommendation confidence: supported, provisional, or insufficient information
- Safeguards against inventing contract terms, overstating utilization as low value, and consolidating without workflow and capacity checks
- Public API rate limiting oriented at bots, not normal human use

**Out of scope (by design):** file uploads, live account connectors, vendor outreach, contract execution, whole-stack audits, and replacing human organizational approval.

---

## Tech stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js 16 (App Router) |
| UI | React 19, TypeScript, Tailwind CSS 4 |
| LLM | OpenAI API. See models below. |
| Streaming | Server-streamed responses into the chat UI |
| Markdown | `react-markdown` |
| Quality | ESLint, focused Node test suites for deadline, commercial-term, assessment, and research-merge behavior |
| Deploy | Vercel |

## Models

The app calls the OpenAI API from the server. The API key stays on the server. Model IDs are environment configuration, not hardcoded, so they can change when OpenAI ships a new generation. The live site currently uses two models, assigned by job:

| Job | Model | Reasoning effort |
|-----|--------|------------------|
| Chat / intake | `gpt-6.1-sol` | `low` |
| Background web research | `gpt-6.1-sol` | `low` |
| Formal assessment | `gpt-6.1-sol` | `medium` (API default) |
| Evidence extraction | `gpt-6-luna` | `medium` (API default) |

On the current OpenAI ladder, Luna (`gpt-6-luna`) is the least expensive and fastest. Sol (`gpt-6.1-sol`) is the middle model, near Astra quality at lower cost. Astra (`gpt-6-astra`) is the slowest, smartest, and most expensive, and this product does not use it. There is no GPT-6 Terra. Reasoning effort is a separate dial from the model. Lower effort favors speed and cost.

Chat and assessment responses stream into the page. Extraction and research run beside the conversation and write into the evidence record. Research is not pasted into the chat. Notice deadlines, runway, and renewal-price math stay in application code, so the model is not doing the date or percentage arithmetic. The footer on the live site names the models in use.

---

## Product principles (summary)

- Help a leader decide on **an existing tool's renewal**, not redesign their entire stack.
- Prefer business value, total cost, operating capacity, and feasible next actions over architecture labels or feature checklists.
- Ask follow-ups only when answers could materially change the recommendation.
- Be explicit about assumptions, unknowns, and provisional status.
- Keep the assessment short enough to read in about a minute.

---

## Portfolio note

Source code, prompts, and the detailed decision methodology are private.

I set the product direction: who it is for, what a useful renewal recommendation has to include, and how the conversation should behave. I did not write the application by hand. The code, prompts, and this write-up were produced with AI coding tools, then reviewed by me.

**Try the live app:** [therenewalsdesk.com](https://www.therenewalsdesk.com)

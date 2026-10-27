# The Renewals Desk

**Live product:** [https://www.therenewalsdesk.com](https://www.therenewalsdesk.com)

The Renewals Desk helps marketing operations and martech leaders decide what to do about one SaaS or marketing technology renewal at a time. Someone reviewing a tool brings the ownership, usage, cost, dependency, and contract facts they have. The desk asks focused follow-ups, then writes a short assessment with a clear next step.

The line on the page: An AI-powered assistant for marketing technology leaders deciding whether to renew, replace, or deprecate a tool or platform in their stack.

This repository is the public portfolio showcase: the product narrative and the architecture. The application source is private.

![The Renewals Desk opening screen: the wordmark, the tagline "An AI-powered assistant for marketing technology leaders deciding whether to renew, replace, or deprecate a tool or platform in their stack", and the opening question asking which tool is under review and which company the user works for](docs/landing.png)

The opening message is a thorough contract renewal review: focused questions, then a written assessment, starting with which tool is under review and which company the user works for. It notes that public employer information like size and industry helps tailor the review. Naming the employer is optional. At set milestones the desk looks up the employer and the vendor from public sources, and keeps asking for what still needs a human answer.

![An intake conversation: the desk asks what the tool is used for and what prompted the review, then asks about cost, contract end date, automatic renewal, and notice](docs/conversation.png)

Questions follow the answers that could change the recommendation. Notice deadlines and the related commercial figures are calculated in application code. The company in the screenshot is fictional.

---

## Problem

A renewal is difficult to call when the evidence is incomplete and split across Marketing Ops, IT, Finance, Procurement, and Legal. Overlapping features, on their own, do not show that a tool can come out safely. The person making the decision needs a timely recommendation that is plain about confidence, tradeoffs, and what still has to be confirmed.

## What it does

The work runs in separate passes: chat intake, evidence extraction, milestone research, and a formal assessment.

1. **Intake conversation.** The chat opens with this:

   > This is a thorough contract renewal review. Focused questions, then a written assessment. To start: which tool are we reviewing, and which company do you work for? Public information about your employer, like its size and industry, helps tailor the review. If you would rather not name your employer, say so.

   Naming the employer is optional. From there the desk gathers purpose, ownership, commercial terms, usage, dependencies, overlap candidates, and vendor context. A follow-up is asked when the answer could change the recommendation. Answers are typed or pasted text.

2. **Structured evidence.** The session holds an evidence record. Each fact has a value, a source, and a state: answered, unknown, conflicting, or not yet asked. A contract term that was never supplied stays unknown.

3. **Public research.** At defined milestones the desk researches the company and the vendor or product, and writes the findings into the evidence record with sources. The chat stays a conversation.

4. **Deadline math.** Notice deadlines and runway are calculated in deterministic application code, using the user's local calendar date, and shown when the timing matters. The language model does not do that arithmetic.

5. **Formal assessment.** A separate assessment pass writes a compact review:
   - **Immediate contract action:** renew, renegotiate, seek an extension, defer, or do not renew
   - **Longer-term direction:** retain, sunset, consolidate, or investigate further
   - **Reasoning and evidence:** the case for the call, with a status of supported, provisional, or insufficient information
   - **Action plan:** owners and next steps

6. **Revision.** New evidence updates the recommendation. Older assessments in the thread are marked superseded.

The session is not saved after the tab closes. Organizational approval stays with the person using the desk.

**Privacy.** The [Privacy and Data](https://www.therenewalsdesk.com/privacy) page describes what happens to a conversation in practice. Storage and logging are turned off for this site's OpenAI requests, so conversations are not kept in this site's OpenAI account, and OpenAI does not use API data to train its models. Under its standard API terms, OpenAI may still keep request content for up to 30 days for abuse monitoring. The site does not have a zero-retention arrangement with OpenAI. The site uses no cookies, tracking, or analytics.

---

## Architecture

```text
┌─────────────────────────────────────────────────────────┐
│  Browser (Next.js App Router UI)                        │
│  • Single-column chat shell                             │
│  • In-memory session evidence and messages              │
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
│  • Commercial terms and notice-deadline calculation     │
│  • Assessment readiness and handoff                     │
│  • Role-separated model calls (chat, extract,           │
│    research, assessment), chosen in configuration       │
└─────────────────────────────────────────────────────────┘
```

**Design choices that shape the product:**

| Choice | Why |
|--------|-----|
| Intake and assessment are separate passes | Questions stay investigative. The written assessment stays compact and consistent. |
| Deadline arithmetic is deterministic | Date math runs in application code, on the user's local calendar date. |
| Research lands in the evidence record | The conversation stays readable, and the recommendation still has sources. |
| Session-only state | One review in the open tab, with no account and no saved transcript. |
| One tool per review | The conversation stays on that renewal. Overlap comes up when it changes the call. |

---

## Features

- A renewal conversation for one B2B marketing technology decision
- A warm desk palette (mahogany and ivory) and a clear brand hierarchy
- Streamed replies for chat and for the assessment
- Markdown rendering for assistant replies
- An in-session evidence record across the intake categories
- Company and vendor research at milestones, stored with sources
- A notice deadline and runway from application code when the inputs are known
- A four-section assessment with copy to clipboard
- A plain status on the recommendation: supported, provisional, or insufficient information
- Safeguards on invented contract terms, on reading low utilization as low value, and on consolidation before workflow and capacity are checked
- Bot-focused rate limits on the public API routes, high enough for a person reviewing many tools

**Outside the product:** file uploads, live account connectors, vendor outreach, contract execution, whole-stack audits, and replacing organizational approval.

---

## Tech stack

| Layer | Technology |
|-------|------------|
| Framework | Next.js 16 (App Router) |
| UI | React 19, TypeScript, Tailwind CSS 4 |
| Models | OpenAI API. Chat, research, and assessment use GPT-6.1 Sol. Extraction uses GPT-6 Luna. GPT-6 Astra is unused unless chosen for a role. Reasoning effort is a separate dial. Requests set `store: false`. |
| Streaming | Server-streamed responses into the chat |
| Markdown | `react-markdown` |
| Quality | ESLint, and Node tests for deadlines, commercial terms, assessment copy, and research merge |
| Deploy | Vercel |

---

## Product principles

- Help a leader decide on one existing tool's renewal.
- Weigh business value, total cost, operating capacity, and a feasible next action ahead of architecture labels or a feature checklist.
- Ask a follow-up only when the answer could materially change the recommendation.
- State assumptions, unknowns, and a provisional status in everyday language.
- Keep the assessment short enough to read in about a minute.

---

## Portfolio note

Source code, prompts, and the detailed decision method are proprietary. They are not in this showcase.

Drafted with AI from the product and the codebase, then reviewed by a human. This is AI-assisted copy.

**Try the live app:** [therenewalsdesk.com](https://www.therenewalsdesk.com)

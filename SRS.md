# SalesGenie AI — Software Requirements Specification

**Status:** Review and sign-off candidate
**Version:** 0.1
**Prepared:** 15 October 2026
**Source:** Project brief, milestone plan, execution/training guides, and the updated 12–13 October task rows

> This document is prepared for stakeholder review. It is not approved or signed on behalf of the team. Approval fields are intentionally blank.

## 1. Purpose

SalesGenie AI is an AI-assisted sales platform for managing prospects, preparing sales outreach, analyzing leads, summarizing sales conversations, and monitoring sales activity. It is intended to reduce repetitive research and administration while helping sales users prioritize and follow up with prospects.

This SRS records the agreed project scope and provides a basis for API design, implementation, and acceptance testing. Milestone 1 is the first implementation focus; the requirements below describe the full six-module project.

## 2. Project objectives

- Maintain a centralized, searchable lead and prospect database.
- Generate company and lead insights linked to the correct lead.
- Support personalized outreach and lead qualification.
- Summarize sales conversations and capture action items.
- Provide sales-pipeline and activity analytics.
- Expose backend functionality through a documented API and a Streamlit user interface.

## 3. Users and operating context

### 3.1 User

- **Sales user:** manages prospects, reviews insights, prepares outreach, records or summarizes conversations, and checks sales activity.

### 3.2 System components

- **User interface:** Streamlit dashboard.
- **API:** Python FastAPI service with Pydantic request/response validation.
- **Persistence:** SQLAlchemy ORM and Alembic migrations; SQLite for local development and PostgreSQL for production, following the selected training-guide baseline.
- **AI service:** OpenAI API, with the model selected through configuration. The requester reports `gpt-4o-mini` as the team-selected model; live-provider acceptance remains to be performed.

The application must not be exposed to the public internet with real contact data until authentication, authorization, deployment security, and operational controls are implemented.

## 4. Scope

### 4.1 Included modules

1. **Lead Management & Prospect Database** — create, import, view, search, update, and track leads.
2. **Lead Intelligence & Company Analysis** — analyze company information and save retrievable insights against a lead.
3. **AI Outreach Generation** — draft personalized emails and sales messages from lead context.
4. **Lead Scoring & Recommendation Engine** — calculate a qualification/likelihood score and suggest next actions.
5. **Conversation Intelligence & CRM Integration** — summarize calls/meetings, extract actions, and support CRM synchronization.
6. **Dashboard & Sales Analytics** — present lead, pipeline, conversion, and outreach activity measures.

### 4.2 Milestone 1 focus

The first milestone is the lead-to-insight workflow:

1. Create or import a prospect.
2. Retrieve, search, and update the prospect.
3. Analyze company information.
4. Persist the resulting insights against the correct lead.
5. Display the lead and saved insights in the Leads screen.

CSV/manual import, duplicate identification, basic qualification information, test evidence, and a demo path are included in the milestone plan. The sheet's Milestone 1 acceptance target is at least 95% manageable records with accurate insights; evaluation requires an agreed test set and insight-review rubric. The separate 85% scoring-accuracy target belongs to Milestone 2 and is not a substitute for the Milestone 1 target.

### 4.3 Excluded from the first milestone

- Production-grade authentication, role-based access control, and multi-tenancy.
- Live CRM synchronization or a commitment to a specific CRM vendor.
- Unattended or autonomous outreach delivery.
- A guarantee that generated company facts are accurate without supplied or verifiable sources.
- The Milestone 2 scoring accuracy target and later milestone outcomes.

## 5. Functional requirements

### 5.1 Lead management

- **FR-LEAD-001:** The system shall create a lead with a company name and optional industry, contact name, email, and phone.
- **FR-LEAD-002:** The system shall assign a unique lead ID and creation/update timestamps.
- **FR-LEAD-003:** The system shall list and retrieve leads, including paginated search/filter results.
- **FR-LEAD-004:** The system shall update lead details and lifecycle status.
- **FR-LEAD-005:** The system shall accept a CSV of leads and report duplicate and invalid rows without silently discarding row errors; the selected Milestone 1 behavior is all-or-nothing, so no rows are imported when any data row is invalid or duplicated.
- **FR-LEAD-006:** The system shall identify likely duplicates using the selected draft rule: normalized email when present; otherwise normalized company name plus contact name. Formal contract sign-off remains recorded below.
- **FR-LEAD-007:** Invalid input and unknown lead IDs shall produce clear, user-visible errors and shall not crash the application.

Selected lifecycle statuses: `New`, `Contacted`, `Qualified`, `Proposal`, `Won`, and `Lost`. Formal contract sign-off remains recorded below.

### 5.2 Company intelligence

- **FR-INTEL-001:** A user shall be able to request company analysis for an existing lead.
- **FR-INTEL-002:** An analysis shall be linked to the lead ID used for the request.
- **FR-INTEL-003:** The system shall save and retrieve analysis results after the request completes.
- **FR-INTEL-004:** An analysis shall return a concise summary, potential needs, opportunities, and a transparent Milestone 1 qualification estimate with its rationale and contributing factors.
- **FR-INTEL-005:** The UI shall show saved analysis alongside the corresponding lead.
- **FR-INTEL-006:** The system shall distinguish model-generated content from verified facts and shall not represent unsupported claims as confirmed facts.

### 5.3 Outreach generation

- **FR-OUT-001:** A user shall be able to request a draft email or sales message using a lead and selected context.
- **FR-OUT-002:** The system shall return a draft for user review; it shall not send messages automatically in the project scope described here.

### 5.4 Lead scoring and recommendations

- **FR-SCORE-001:** The system shall produce a lead score and explain the principal factors used.
- **FR-SCORE-002:** The system shall provide a next-best-action recommendation when sufficient lead/activity data is available.
- **FR-SCORE-003:** Scores shall be labeled as estimates, not guarantees of conversion.

### 5.5 Conversation intelligence and CRM integration

- **FR-CONV-001:** A user shall be able to submit a call/meeting transcript for summarization.
- **FR-CONV-002:** The system shall return a summary, key points, and action items.
- **FR-CRM-001:** The system shall support synchronization with an agreed CRM through documented integration contracts. The CRM vendor and synchronization scope remain open decisions.

### 5.6 Dashboard and analytics

- **FR-DASH-001:** The dashboard shall show total leads and lead counts by lifecycle status.
- **FR-DASH-002:** As relevant data becomes available, the dashboard shall show pipeline, conversion, and outreach activity measures.
- **FR-DASH-003:** Analytics shall be derived from persisted application data and shall not fabricate unavailable measures.

## 6. Data requirements

The initial logical data model is:

| Entity | Purpose | Key relationships |
|---|---|---|
| Lead | Company/contact details and lifecycle status | One lead may have many insights, scores, outreach drafts, activities, and conversation summaries |
| CompanyInsight | Persisted company analysis and qualification information | Belongs to one lead |
| LeadScore | Score, rationale, and scoring timestamp | Belongs to one lead |
| OutreachDraft | Generated message, type, and creation timestamp | Belongs to one lead |
| Activity | Lead engagement history and event details | Belongs to one lead |
| ConversationSummary | Summary, key points, and action items | Belongs to one lead when the conversation is associated with a lead |
| ImportBatch | CSV import counts and row-level validation errors | Tracks one import request; imported leads remain in Lead |
| AgentRun | Workflow name, status, timing, and safe error/checkpoint metadata | May reference a lead; does not store prompts, credentials, or full agent state |

The implemented draft database schema contains eight tables: the six business entities above plus `ImportBatch` and `AgentRun` operational metadata. The team must still approve the column types, nullability, indexes, and migration strategy before production use. The initial lead fields are `lead_id`, `company_name`, `industry`, `contact_name`, `email`, `phone`, `lead_status`, `created_at`, and `updated_at`.

## 7. Non-functional requirements

- **NFR-REL-001:** At least 95% of the agreed Milestone 1 prospect test set shall be manageable through the supported create/import, retrieve, and update workflow, and generated insights shall meet the team's agreed accuracy rubric. The test report shall record the test-set definition, numerator, denominator, rubric, reviewer outcome, and failures. Deterministic test-provider results do not establish real insight accuracy.
- **NFR-REL-002:** Missing IDs, invalid records, duplicate imports, and provider/database failures shall produce explicit errors; they shall not be reported as successful operations.
- **NFR-SEC-001:** API keys and populated environment files shall remain outside version control. Use a documented `.env.example` containing placeholders only.
- **NFR-SEC-002:** Demo and test runs shall use synthetic or approved sample records, not personal or confidential customer data.
- **NFR-SEC-003:** Local services shall bind to localhost by default. Production exposure requires a separately reviewed security design.
- **NFR-DATA-001:** A successful analysis shall persist with a valid lead foreign key and be retrievable after the request.
- **NFR-USE-001:** The user shall be able to complete the demo flow: Add Lead → retrieve lead → Analyze Company → view saved insights.
- **NFR-PERF-001:** The project brief's under-five-second analysis/outreach target is a later evaluation goal; it must be measured and reported under representative conditions before being claimed.
- **NFR-MAINT-001:** API request/response shapes and shared data fields shall be documented and agreed by backend and database owners before cross-module implementation.

## 8. Proposed acceptance criteria

Milestone 1 is acceptable when:

1. The agreed test set and results are recorded; at least 95% meet the agreed manageable-record criterion, and generated insights meet the agreed human-review accuracy rubric.
2. Invalid rows and duplicate imports are reported clearly.
3. Analysis is linked to the correct lead, persists, and can be retrieved.
4. All team members can follow the agreed local setup instructions.
5. The team can demonstrate the complete lead-to-saved-insight flow using approved sample data.
6. Open critical defects, their owners, and their disposition are included in the test/sign-off report.

## 9. Assumptions and decisions requiring team confirmation

| Decision | Current baseline | Confirmation needed |
|---|---|---|
| Core stack | Python, FastAPI, Streamlit, Pydantic, SQLAlchemy, Alembic | Implemented dependency ranges are in `requirements.txt`; approve/lock exact versions |
| Database | SQLite for local development; PostgreSQL for production | Confirm whether CI/tests must also run against PostgreSQL |
| AI provider/model | OpenAI API; requester reports team selection of `gpt-4o-mini` (the execution guide also mentions Groq) | Live key configuration, provider run, and final API/provider policy review; never place a real key in Git |
| Lead status values | New, Contacted, Qualified, Proposal, Won, Lost | Requester reports team approval of selected values; formal SRS sign-off remains pending |
| Duplicate rule | Normalized email; otherwise company + contact | Requester reports team approval of selected rule; formal SRS/API sign-off remains pending |
| Milestone 1 accuracy acceptance | Sheet requires >=95% manageable records and accurate insights | Agree test-set denominator and reviewer rubric; run and record real evaluation before claiming acceptance |
| CRM integration | Not committed to a vendor | Select vendor and synchronization scope in a later milestone |
| Authentication | Excluded from initial local milestone | Required before production deployment |
| Repository | `https://github.com/230701224/sales-genie-ai.git` supplied by requester | Confirm access and the actual local source folder before repo work |

## 10. Review and sign-off

Signing indicates agreement with the scope, requirements, and open-decision handling above. It does not claim that implementation or testing is complete.

| Role | Name | Decision (Approve / Changes requested) | Date | Signature / approval reference |
|---|---|---|---|---|
| Project Lead |  |  |  |  |
| Backend Developer |  |  |  |  |
| Database/DevOps Engineer |  |  |  |  |
| Full Team representative(s) |  |  |  |  |

## 11. Reference material

- [AI Sales Intelligence Platform.pdf](./AI%20Sales%20Intelligence%20Platform.pdf)
- [SalesGenie - Milestone 1 Plan.pdf](./SalesGenie%20-%20Milestone%201%20Plan.pdf)
- [SalesGenie - execution guide.pdf](./SalesGenie%20-%20execution%20guide.pdf)
- [SalesGenie AI Intern training guide.pdf](./SalesGenie%20AI%20Intern%20training%20guide.pdf)
- [SalesGenie AI – Complete Installation-Guide.pdf](./SalesGenie%20AI%20%E2%80%93%20Complete%20Installation-Guide.pdf)

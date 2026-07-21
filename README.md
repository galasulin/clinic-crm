# 🏥 Clinic CRM / EMR — Project Showcase

A bilingual (**Hebrew RTL / English**) clinic management platform I designed and built — full-stack, engine-driven, multi-tenant, and now telephony- and AI-enabled.

> 🔒 **This is a public showcase, not the source.** The code and any real data are kept private. Everything here is a high-level overview; all screenshots use **demo data only**.

![.NET 10](https://img.shields.io/badge/.NET-10%20LTS-512BD4?logo=dotnet&logoColor=white)
![React 19](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?logo=microsoftsqlserver&logoColor=white)
![SignalR](https://img.shields.io/badge/SignalR-realtime-0078D7)
![Tested](https://img.shields.io/badge/tests-xUnit-5E2750?logo=xunit&logoColor=white)
![Status](https://img.shields.io/badge/status-active%20development-success)

---

## 📋 Overview / סקירה

🇬🇧 A management system for a physiotherapy / clinic setting: scheduling, patient records, a smart **intake → clinical narrative** engine, a leads CRM with a **sales pipeline**, quotes/orders + billing, inventory & contracts, a **contact-center / telephony (CTI)** layer, a patient self-service portal, and a **self-service reporting & analytics platform** (report anything, schedule it, share it, ask it in plain language) — with an **AI assist layer** on top. The guiding idea is **engines over screens** — most features are *composed* from a form builder, lookup data and a narrative engine, instead of being hard-coded. It runs **real-time** (live calendar + push notifications over WebSockets), includes a **no-code automation** builder, and is packaged to run **on-premises as a single Windows Service** against a dedicated SQL server.

🇮🇱 מערכת ניהול למרפאה: יומן תורים, תיק מטופל, מנוע **שאלון חכם → אנמנזה קלינית**, CRM לידים עם **משפך מכירות**, הצעות מחיר/הזמנות וחיוב, מלאי וחוזים, שכבת **מוקד טלפוני (CTI)**, פורטל מטופלים לשירות עצמי, ו**פלטפורמת דוחות ואנליטיקה** (דיווח על כל נתון, תזמון, שיתוף, ו"שאל את הנתונים" בשפה חופשית) — עם שכבת **AI** מעל הכל. הרעיון המנחה — **מנועים במקום מסכים**. המערכת עובדת ב**זמן-אמת**, כוללת מנוע **אוטומציות ללא-קוד**, וארוזה לרוץ **On-Prem כשירות Windows יחיד** מול שרת SQL ייעודי.

---

## 🏗️ Architecture

![Architecture](assets/architecture.svg)

Clean Architecture split into **Api / Core / Infrastructure**, a React SPA on top, SQL Server underneath, and background jobs + signed webhooks for outbound integrations. Auth is JWT with refresh-token rotation; the patient portal uses a **separate, patient-scoped token** that is rejected by staff endpoints. Tenancy is enforced per `ClinicId`. External integrations (telephony, AI, messaging, lead sources) are behind **provider-agnostic adapters**, each with a built-in "dev" simulator so the whole system runs end-to-end without any third-party account.

---

## 🧩 Modules

| Domain | Highlights |
| ------ | ---------- |
| **Engines** | Form Builder (incl. a repeatable **"add-row" table field**) · Static Data (lookups) · Narrative engine (structured answers → Hebrew clinical text with conditionals) |
| **Patients** | Full medical record, visits, collapsible intake sections, smart questionnaires, mailing-consent & referral panels, multi-signature **reception / intake forms** (admin-locked once signed), a **patient-relations** tab (linked family / related records), and a **unified activity timeline** that merges appointments, invoices, service cases, logged calls/messages, tasks and documents into one chronological, per-source-filterable feed |
| **Scheduling** | Multi-clinic / rooms / practitioners; daily / weekly (per-therapist) / monthly views; **live updates across users**; per-branch work hours + one-off exceptions; drag-move & drag-resize of appointments **and** blocks; visual allocations defined on the grid; preliminary-check (PTA) columns with computed arrival time; recurring series; **waitlist**; **two-way SMS** confirm/cancel; a new appointment's **branch auto-fills from the patient's recorded treating branch** (editable, with a mismatch warning); full Excel export by practitioner & date range |
| **Tasks & Automation** | Multi-assignee tasks, transfer, dependencies (A blocks B), SLA timers; a **no-code automation builder** (triggers → conditions → actions: reminders, escalation to a team manager, weighted round-robin routing, lead scoring). The engine is **stateful**: an action can carry a **time delay** ("wait N days, then…") — deferred with a frozen snapshot of the trigger context and executed later by a background tick, exactly once. **Commercial-pipeline triggers** (invoice issued / paid / overdue, low-satisfaction survey) turn billing and feedback events into follow-up work automatically |
| **Real-time** | System-wide live sync over SignalR: a single EF SaveChanges interceptor broadcasts every entity change to connected clients, so cards, lists, the calendar and forms refresh instantly across users — no polling, no per-screen wiring. Overlapping fields (e.g. a patient-card box and its questionnaire) stay in sync both ways from one source of truth |
| **CRM — Leads** | Configurable statuses / sources / fields, public intake API, duplicate detection **and a smart merge** (same phone in any format / same email → one survivor absorbs the timeline, tasks, quotes and empty fields; the duplicates land in the recycle bin, never lost), activity timeline, campaigns. A **drag-and-drop Kanban board** (columns = pipeline stages, dragging a card changes the stage and fires the matching automations) and **bulk actions** on the list (multi-select → reassign / set stage / export / recycle) |
| **Saved Views** | A reusable, per-screen **list-view engine** — capture the current filters + search + sort as a named preset, keep it private or **share it with the whole team**, and pin one as your default. Generic across list screens (leads, service cases, and more) |
| **Sales** | Quote → order → invoice document flow with **server-side pricing** (line + document discounts + VAT, recomputed on every save), per-year numbering, branded printable/PDF quotes; a **discount-approval workflow** (over-threshold discounts can't be sent until a manager approves — and raising the discount afterwards voids the approval); **guided selling** (per-stage checklists + next-best-action hints); a **weighted sales forecast** — pipeline value × stage probability by month / rep / stage; **monthly sales targets (quotas)** per rep with live attainment bars and one-click carry-over |
| **Service Desk (Cases)** | Numbered service tickets with an **SLA engine**: first-response and resolution targets derived from priority, live countdown chips, the first staff note stamps the response automatically, and a periodic sweep flags breaches exactly once — real-time alert to the assignee + an automation trigger for escalation (e.g. SMS the manager). Full timeline, status flow with reopen, linked to the patient/lead. Worked from a list **or a drag-and-drop Kanban board** with SLA-breach badges |
| **Omni-Channel Inbox** | Every inbound SMS / WhatsApp lands on **one conversation thread per phone number** (any formatting), auto-matched to the patient or lead. A two-pane chat UI with unread badges, agent assignment, inline reply through the right channel, close/reopen, and a jump to the patient file. An automation trigger fires on every inbound message — the hook for an auto-reply agent |
| **Email Campaigns** | Consent-enforced bulk email (audience filters, dedup, personalization) sent by a throttled background job, with **per-recipient open / click / unsubscribe tracking** (pixel + wrapped links) and a branded unsubscribe page that flips the contact's marketing consent off — compliance first. Live funnel chips on each campaign card |
| **Knowledge Base** | Searchable internal articles (procedures, call scripts) with categories, tags and view counts; articles marked "publish" surface on the patient portal as an FAQ via a PHI-free public endpoint |
| **Telephony / Contact Center (CTI)** | Provider-agnostic PBX integration: **screen-pop** on inbound calls (caller matched to patient/lead, one-click jump to the card), **click-to-call** from anywhere, a **live wallboard** of active calls, agent **queue login/logout** + break statuses, **call-recording mute** for sensitive data, **historical call-log sync** (idempotent), a **predictive-dialer** upload from the call queue, and **smart IVR routing** (recognized callers routed by an external-layer webhook with their name attached). Every call is journaled and linked to the patient/lead timeline |
| **AI Assist** | A generative layer behind a **pluggable multi-provider gateway** (three interchangeable LLM vendors + a dev simulator; provider, per-provider keys and model picked in the admin UI at runtime): message drafting, one-click lead & patient summaries, an **insights engine** (period-vs-period KPIs, auto findings with recommendations, an AI executive narrative and conversational "ask your data" grounded strictly in the clinic's numbers) and a **weekly executive digest email**; a **Next-Best-Action** panel on the patient record — a local **churn-risk** score computed from retention signals (visit recency, no-shows, open cases, unpaid balance, latest satisfaction) plus an AI-ranked list of concrete next steps, with a graceful heuristic fallback when no AI provider is configured; satisfaction-survey **sentiment routing** (a detractor auto-opens a recovery task / alerts a manager; a promoter can trigger a referral ask); **plus local predictive models trained on the clinic's own data** — lead scoring with human-readable reasons and appointment **no-show risk** — that need no external service and refresh nightly |
| **Voice of Customer (NPS)** | Tokenized **no-login survey pages** sent by SMS / email from any patient or lead card (or via automations); an NPS dashboard with the promoter / passive / detractor split, monthly trend and recent comments |
| **Business Partners** | Unified ERP-style card (customer / supplier / lead), lead→customer conversion, **tiered / per-customer price lists** & discounts |
| **Finance** | Procurement cycle + supplier **A/P** balance; a customer **A/R aging** report (0–30 / 31–60 / 61–90 / 90+ buckets); invoices carry a compliant tax-authority **allocation number** and render to a **server-side PDF** |
| **Inventory** | Products, warehouses, **stock movements** and low-stock alerts |
| **Contracts & Subscriptions** | Treatment packages / subscriptions with **automatic renewal billing** and package-utilization enforcement, driven by a nightly sweep |
| **Documents** | Templates, generation, multi-party **tamper-evident digital signatures** (SHA-256 fingerprint over the document + signature + signer + trusted server timestamp, with a one-click *verify* that detects any post-signing edit), brand-framed forms rendered to **real PDF** client-side (and emailed as an attachment) |
| **Patient Portal** | OTP login, **self-booking of free slots**, remote questionnaire filling, personal dashboard, staff "send to portal" (SMS / WhatsApp), remote signing |
| **Reporting & Analytics** | A self-service **report builder over every value in the system** — built-in entity columns, custom fields, and *questionnaire* answers (including "add-row" table columns and multi-selects, exploded on the fly) are all groupable / filterable / aggregatable, with Hebrew labels on every field. Eight chart types plus a **two-dimensional pivot**; **time-bucketing** (day / week / month / quarter / year) with **period-over-period Δ%**; **multi-measure series** on one chart; **drill-through** from any aggregate straight to the underlying records; live auto-preview; Excel / CSV export; and a guarded **read-only SQL power tool** for admins (SELECT-only, sandboxed in an always-rolled-back transaction, row/time capped, fully audited). Saved reports can be **emailed on a schedule**, fire **threshold alerts** ("email me if new leads < 10 this week"), be **shared by public link / embedded via iframe**, and be arranged on a **drag-and-drop dashboard** (reorder + resize, auto-saved). **View-access is governed per department** (each person sees only the report departments they're granted), and an **AI "ask your data"** box turns a plain-language question into a ready-built report. Alongside it: a **live** manager KPI dashboard (funnel, occupancy, no-show rate, task load) that auto-refreshes over WebSockets, pinned widgets, and a user-activity report |
| **Messaging** | System-wide, in-app-**editable SMS & email templates** with merge tokens, referenced by the automation engine (edit the wording once, every rule updates); **branded, deliverability-focused emails** (inline-CSS RTL HTML + a plain-text alternative + Reply-To to lower spam scores) |
| **Growth** | Hosted landing-page builder → leads, ad connectors, HMAC-signed outbound webhooks |
| **Admin & Security** | Visual **granular permission tree** (module → tab → field, allow/deny per role / group / user, deny-by-default for sensitive actions); user / role / group management; **opt-in 2FA** (TOTP + email OTP); **idle session auto-lock**; a **security / sign-in event log** that records the real reason each login succeeded or failed (the login screen stays generic to prevent account enumeration); editable communication / telephony / AI / retention / directory settings |
| **Design system & UX** | A single **token layer** (color / type / radius / shadow / motion) drives every screen in **light and dark** — no hard-coded values, enforced by lint; a library of design-system primitives; full **RTL/LTR** via logical CSS properties; and a **liquid-glass / glassmorphism sign-in** experience (animated aurora, frosted panels, reduced-motion-aware) |

---

## 🛠️ Tech & Engineering

- **Backend:** ASP.NET Core Web API (**.NET 10 LTS**), EF Core, SQL Server, JWT + refresh rotation, RBAC policies, **SignalR** (WebSockets) for real-time push, **Hangfire** background jobs (reminders, escalation, retention, scheduled report emails, nightly AI scoring).
- **Frontend:** React 19 + TypeScript + Vite + MUI (full RTL) + react-i18next (he/en live switch), styled by a **token-driven design system** (a single design-token layer feeding a themed component set: dark navigation rail, semantic status colors, enterprise-clean cards/tables — so every new screen inherits the language automatically), with a **global quick-jump search** in the top bar and a synonym-aware admin search across every settings screen.
- **Integrations:** provider-agnostic adapters for telephony (CTI), AI, SMS / email / WhatsApp, and lead sources — each with a "dev" simulator, so every flow is demoable and testable without a third-party account.
- **Cross-cutting:** universal audit, multi-tenant isolation, background job queue with retries, deny-by-default capability checks down to the field level.
- **Data-safety by design:** nothing is ever hard-deleted in normal use — a global soft-delete turns every delete into a recoverable state, surfaced in an **admin recycle bin** (who deleted what, when, with full field values and one-click restore). Destructive actions use a type-to-confirm safeguard and an inline **undo**; the single true "purge" is a dedicated, permission-gated action that only ever removes rows already in the bin.
- **Production hardening (on-prem ready):** ships as a **Windows Service** that serves the API *and* the built SPA from one process/port; **structured rolling file logs**; a **global exception handler** (full detail to the log, a clean generic error to the client); **rate-limited** sign-in / OTP endpoints (brute-force protection); a **deep health check** that verifies DB connectivity; and **EF transient-fault retry** for resilience against a dedicated, separate SQL server. One-command publish script + a full deployment runbook.
- **Quality:** GitHub Actions CI (build + tests for backend, build for frontend), EF migrations, and a growing **xUnit test suite** (EF In-Memory) covering signature integrity & tamper detection, server-side money math and the discount-approval gate, telephony webhooks (caller matching, idempotency, secret gate, IVR routing), NPS scoring, weighted-forecast and quota-attainment math, the insights engine, duplicate-merge contract, case SLA engine, inbox threading/normalization, and campaign consent/tracking/unsubscribe compliance.

---

## 🖼️ Screenshots

> Demo data only — no real patients.

| | |
| --- | --- |
| _Add `assets/01-dashboard.png`_ | _Add `assets/02-calendar.png`_ |

<!-- Drop demo-data screenshots into assets/ and reference them here, e.g.:
![Dashboard](assets/01-dashboard.png)
-->

---

## 🔐 Privacy & scope

This repository intentionally contains **no source code, no schema, no secrets, and no real data**. It exists to document the project and my role in building it. Implementation details are available on request in a suitable setting.

---

<sub>Built and maintained by <a href="https://github.com/galasulin">@galasulin</a>.</sub>

# 🏥 Clinic CRM / EMR — Project Showcase

A bilingual (**Hebrew RTL / English**) clinic management platform I designed and built — full-stack, engine-driven, multi-tenant, and now telephony- and AI-enabled.

> 🔒 **This is a public showcase, not the source.** The code and any real data are kept private. Everything here is a high-level overview; all screenshots use **demo data only**.

![.NET 8](https://img.shields.io/badge/.NET-8-512BD4?logo=dotnet&logoColor=white)
![React 19](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?logo=microsoftsqlserver&logoColor=white)
![SignalR](https://img.shields.io/badge/SignalR-realtime-0078D7)
![Tested](https://img.shields.io/badge/tests-xUnit-5E2750?logo=xunit&logoColor=white)
![Status](https://img.shields.io/badge/status-active%20development-success)

---

## 📋 Overview / סקירה

🇬🇧 A management system for a physiotherapy / clinic setting: scheduling, patient records, a smart **intake → clinical narrative** engine, a leads CRM with a **sales pipeline**, quotes/orders + billing, a **contact-center / telephony (CTI)** layer, a patient self-service portal, and reporting — with an **AI assist layer** on top. The guiding idea is **engines over screens** — most features are *composed* from a form builder, lookup data and a narrative engine, instead of being hard-coded. It runs **real-time** (live calendar + push notifications over WebSockets), includes a **no-code automation** builder, and is packaged to run **on-premises as a single Windows Service** against a dedicated SQL server.

🇮🇱 מערכת ניהול למרפאה: יומן תורים, תיק מטופל, מנוע **שאלון חכם → אנמנזה קלינית**, CRM לידים עם **משפך מכירות**, הצעות מחיר/הזמנות וחיוב, שכבת **מוקד טלפוני (CTI)**, פורטל מטופלים לשירות עצמי, ודוחות — עם שכבת **AI** מעל הכל. הרעיון המנחה — **מנועים במקום מסכים**. המערכת עובדת ב**זמן-אמת**, כוללת מנוע **אוטומציות ללא-קוד**, וארוזה לרוץ **On-Prem כשירות Windows יחיד** מול שרת SQL ייעודי.

---

## 🏗️ Architecture

![Architecture](assets/architecture.svg)

Clean Architecture split into **Api / Core / Infrastructure**, a React SPA on top, SQL Server underneath, and background jobs + signed webhooks for outbound integrations. Auth is JWT with refresh-token rotation; the patient portal uses a **separate, patient-scoped token** that is rejected by staff endpoints. Tenancy is enforced per `ClinicId`. External integrations (telephony, AI, messaging, lead sources) are behind **provider-agnostic adapters**, each with a built-in "dev" simulator so the whole system runs end-to-end without any third-party account.

---

## 🧩 Modules

| Domain | Highlights |
| ------ | ---------- |
| **Engines** | Form Builder · Static Data (lookups) · Narrative engine (structured answers → Hebrew clinical text with conditionals) |
| **Patients** | Full medical record, visits, collapsible intake sections, smart questionnaires, mailing-consent & referral panels |
| **Scheduling** | Multi-clinic / rooms / practitioners; daily / weekly (per-therapist) / monthly views; **live updates across users**; per-branch work hours + one-off exceptions; drag-move & drag-resize of appointments **and** blocks; visual allocations defined on the grid; preliminary-check (PTA) columns with computed arrival time; recurring series; **waitlist**; **two-way SMS** confirm/cancel; full Excel export by practitioner & date range |
| **Tasks & Automation** | Multi-assignee tasks, transfer, dependencies (A blocks B), SLA timers; a **no-code automation builder** (triggers → conditions → actions: reminders, escalation to a team manager, weighted round-robin routing, lead scoring) |
| **Real-time** | System-wide live sync over SignalR: a single EF SaveChanges interceptor broadcasts every entity change to connected clients, so cards, lists, the calendar and forms refresh instantly across users — no polling, no per-screen wiring. Overlapping fields (e.g. a patient-card box and its questionnaire) stay in sync both ways from one source of truth |
| **CRM — Leads** | Configurable statuses / sources / fields, public intake API, duplicate detection, activity timeline, saved views, campaigns |
| **Sales** | Quote → order → invoice document flow with **server-side pricing** (line + document discounts + VAT, recomputed on every save), per-year numbering, branded printable/PDF quotes; **guided selling** (per-stage checklists + rule-based next-best-action hints on each lead); a **weighted sales forecast** — pipeline value × stage probability, bucketed by month, rep and stage |
| **Telephony / Contact Center (CTI)** | Provider-agnostic PBX integration: **screen-pop** on inbound calls (caller matched to patient/lead, one-click jump to the card), **click-to-call** from anywhere, a **live wallboard** of active calls, agent **queue login/logout** + break statuses, **call-recording mute** for sensitive data, **historical call-log sync** (idempotent), a **predictive-dialer** upload from the call queue, and **smart IVR routing** (recognized callers routed by an external-layer webhook with their name attached). Every call is journaled and linked to the patient/lead timeline |
| **AI Assist** | A generative layer (message drafting with tone/channel control, one-click lead & patient summaries) behind a pluggable provider, **plus local predictive models trained on the clinic's own data** — lead scoring with human-readable reasons, and appointment **no-show risk** — that need no external service and refresh nightly |
| **Voice of Customer (NPS)** | Tokenized **no-login survey pages** sent by SMS / email from any patient or lead card (or via automations); an NPS dashboard with the promoter / passive / detractor split, monthly trend and recent comments |
| **Business Partners** | Unified ERP-style card (customer / supplier / lead), lead→customer conversion, price lists & discounts |
| **Finance** | Procurement cycle + supplier A/P balance |
| **Documents** | Templates, generation, multi-party **tamper-evident digital signatures** (SHA-256 fingerprint over the document + signature + signer + trusted server timestamp, with a one-click *verify* that detects any post-signing edit), brand-framed forms rendered to **real PDF** client-side (and emailed as an attachment) |
| **Patient Portal** | OTP login, **self-booking of free slots**, remote questionnaire filling, personal dashboard, staff "send to portal" (SMS / WhatsApp), remote signing |
| **Reporting** | Report builder (group-by / aggregate / filters), Excel import & export, a **live** manager KPI dashboard (funnel, occupancy, no-show rate, task load — auto-refreshing over SignalR), pinned dashboard widgets, a user-activity report + CSV export |
| **Messaging** | System-wide, in-app-**editable SMS & email templates** with merge tokens, referenced by the automation engine (edit the wording once, every rule updates); **branded, deliverability-focused emails** (inline-CSS RTL HTML + a plain-text alternative + Reply-To to lower spam scores) |
| **Growth** | Hosted landing-page builder → leads, ad connectors, HMAC-signed outbound webhooks |
| **Admin & Security** | Visual **granular permission tree** (module → tab → field, allow/deny per role / group / user, deny-by-default for sensitive actions); user / role / group management; **opt-in 2FA** (TOTP + email OTP); **idle session auto-lock**; a **security / sign-in event log** that records the real reason each login succeeded or failed (the login screen stays generic to prevent account enumeration); editable communication / telephony / AI / retention / directory settings |

---

## 🛠️ Tech & Engineering

- **Backend:** ASP.NET Core 8 Web API, EF Core, SQL Server, JWT + refresh rotation, RBAC policies, **SignalR** (WebSockets) for real-time push, **Hangfire** background jobs (reminders, escalation, retention, nightly AI scoring).
- **Frontend:** React 19 + TypeScript + Vite + MUI (full RTL) + react-i18next (he/en live switch).
- **Integrations:** provider-agnostic adapters for telephony (CTI), AI, SMS / email / WhatsApp, and lead sources — each with a "dev" simulator, so every flow is demoable and testable without a third-party account.
- **Cross-cutting:** universal audit, multi-tenant isolation, background job queue with retries, deny-by-default capability checks down to the field level.
- **Data-safety by design:** nothing is ever hard-deleted in normal use — a global soft-delete turns every delete into a recoverable state, surfaced in an **admin recycle bin** (who deleted what, when, with full field values and one-click restore). Destructive actions use a type-to-confirm safeguard and an inline **undo**; the single true "purge" is a dedicated, permission-gated action that only ever removes rows already in the bin.
- **Production hardening (on-prem ready):** ships as a **Windows Service** that serves the API *and* the built SPA from one process/port; **structured rolling file logs**; a **global exception handler** (full detail to the log, a clean generic error to the client); **rate-limited** sign-in / OTP endpoints (brute-force protection); a **deep health check** that verifies DB connectivity; and **EF transient-fault retry** for resilience against a dedicated, separate SQL server. One-command publish script + a full deployment runbook.
- **Quality:** GitHub Actions CI (build + tests for backend, build for frontend), EF migrations, and an **xUnit test suite** (EF In-Memory) covering signature integrity & tamper detection, server-side money math and document lifecycle, telephony webhooks (caller matching, idempotency, secret gate, IVR routing), NPS scoring, and the weighted-forecast math.

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

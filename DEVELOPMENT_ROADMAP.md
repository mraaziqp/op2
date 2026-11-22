# Ops Intelligence — Development Roadmap

This project follows a controlled, 7-phase implementation roadmap. Each phase requires passing concrete acceptance criteria before advancing.

---

## Phase 1 — Foundation & Architecture (Current Phase)

**Status:** Implemented

- Application shell with persistent sidebar navigation, contextual top bar, global search, notification drawer, and Light/Dark theme support.
- Strict TypeScript domain models (`src/types/domain.ts`) separating Calls, WorkItems, Activities, FollowUps, Servers, ComplianceFindings, ComplianceSnapshots, KnowledgeArticles, Runbooks, Engineers, ImportBatches, SourceRows, Attachments, and AuditLogs.
- Durable browser persistence via IndexedDB (`src/services/indexedDbService.ts`) + JSON backup/restore.
- Repository layer (`src/services/repository.ts`) with input validation, immutable activity timelines, and automatic audit trail recording.
- Honest integration status reporting (Firebase Cloud marked unconfigured until provisioned).
- Operator session & access boundary scaffolding (local operator identity & role boundaries).
- Clearly labelled demonstration dataset (`isDemo: true`) with one-click seed and purge controls.
- Documentation: `README.md`, `DEVELOPMENT_ROADMAP.md`, `DATA_MODEL.md`.

---

## Phase 2 — Universal Excel Ingestion & Export Engine

**Target:** Safe migration of `WIP 0602.xlsx` and future multi-workbook uploads.

- SheetJS multi-workbook XLSX/CSV parser inspecting irregular layouts, merged cells, and missing headers.
- Automatic sheet classification (`Mohammed Raaziq Parker`, `Calls`, `Frank format`, `Compliance`, `Extra Details`, `Sheet1`/`3`/`5`, `AWS Server Decommission Procedu`, `Umars Handover`, `Notes`, `Sheet2`).
- Interactive column mapping wizard and reusable mapping profiles.
- Conservative date/serial parsing flagging ambiguous dates instead of guessing.
- Splitting stacked cell updates into independent immutable `ActivityRecord` entries while preserving raw cell text.
- Duplicate & conflict reconciliation UI, selective import, incremental updates, `ImportBatch` / `SourceRow` provenance, and batch rollback.
- Export profiles matching original Excel reporting formats.

---

## Phase 3 — Operational Workspace

**Target:** Full daily execution for Calls, Work Log, and Follow-Ups.

- Dedicated structured Call/Ticket entry (`Title`, `Subject`, `Body`, BMC Incident/Change refs, multiple servers).
- Daily operational journal accumulating multi-day activities on work items.
- Follow-up views (`Due today`, `Overdue`, `Due this week`, `Waiting for feedback`, `Waiting for another team`, `Completed`).
- Reopening & resolution history preservation without overwriting prior records.
- Waiting-On blocker queue across teams and engineers.

---

## Phase 4 — Infrastructure & Compliance Centre

**Target:** Connected server profiles, compliance snapshots, and interactive runbooks.

- Searchable Server Explorer (`Extra Details` + linked calls, work items, compliance findings, and timelines).
- Identity disambiguation flagging uncertain hostname matches.
- All 12 compliance categories (`ADDM Stale`, `Computers OU`, `Not ATIII Mirrored`, `ATIII not Mirrored`, `VMware PoweredOff`, `Crowdstrike`, `SCOM`, `SYSMON`, `Non-standard VMware names`, `Non-production CLS vs SLA`, `Production CLS vs SLA`, `ADDM EBX Decom`).
- Historical compliance snapshots, scan comparisons, exception tracking, and recurring-server detection.
- Interactive Runbooks preloaded with the 10-stage AWS Server Decommission Procedure and Umar's Handover.

---

## Phase 5 — AI Operations Assistant & Knowledge Intelligence

**Target:** Evidence-based Gemini retrieval and knowledge extraction.

- Server-side Gemini API integration (`@google/genai`) retrieving over actual workspace records.
- Citation-backed operational Q&A distinguishing confirmed facts, attempted steps, and AI suggestions.
- Explicit insufficient-evidence responses (zero invented references or procedures).
- Universal smart capture box (paste email/Teams/ticket text -> structured proposal requiring user confirmation).
- Knowledge article extraction workflow with human approval before publishing.

---

## Phase 6 — Reporting & Operational Automation

**Target:** Automated report generation and exports.

- Daily, Weekly, Handover, Compliance, Call Summary, Engineer Workload, Server History, and Management reports.
- Editable preview before exporting to PDF, DOCX, and XLSX.
- Scheduled reminder rules and configurable summary templates.

---

## Phase 7 — Production Hardening & Deployment

**Target:** Security, validation, and operational resilience.

- Full Firebase Firestore & Auth integration with hardened security rules (`firestore.rules`) and rules unit tests.
- End-to-end verification of import rollback, activity immutability, backup/restore, and access boundaries.
- Performance profiling and production deployment checklist.

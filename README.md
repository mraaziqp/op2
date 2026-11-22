# Ops Intelligence

Personal IT Infrastructure Operations Command Centre, Reporting, and Knowledge Management System.

Designed to transform historical Excel workbooks (such as `WIP 0602.xlsx`), BMC Remedy/Helix calls, compliance findings, server asset inventories, technical handovers, and operational runbooks into a connected, searchable, and auditable workspace.

---

## Current Phase: Phase 1 — Foundation & Architecture

Phase 1 establishes the application shell, domain data model, IndexedDB local persistence engine, authentication/access boundary scaffolding, immutable activity and audit logging foundation, error handling, and clearly labelled demonstration data controls.

### Persistence Status (Honest Integration Notice)

- **Active Local Persistence:** Browser **IndexedDB** (`ops_intelligence_db`) stores all operational entities durably across page refreshes (`localStorage` is only used for lightweight UI theme preference).
- **Cloud Persistence (Firebase Firestore & Auth):** **Not configured.** During initial setup, Firebase provisioning was declined. The application does not simulate cloud sync or fabricate Firebase connections. See [Database & Cloud Setup](#database--cloud-setup) below to connect a Firebase project.
- **Server-Side AI (Gemini API):** Scheduled for **Phase 5** via server-side API routes using `GEMINI_API_KEY`. No API keys are exposed in client-side code.

---

## Architecture Overview

- **Frontend:** React 19 + TypeScript (strict typing) + Tailwind CSS v4 + Lucide Icons
- **Local Durable Persistence:** Native IndexedDB wrapper (`src/services/indexedDbService.ts`) with full JSON backup & restore
- **Data Access & Audit Layer:** `OpsRepository` (`src/services/repository.ts`) enforcing immutable `ActivityRecord` timelines, automatic `AuditLog` entries, and strict `isDemo` isolation
- **Domain Entities:** 14 normalized entity types in `src/types/domain.ts` (`WorkItem`, `CallRecord`, `ActivityRecord`, `FollowUp`, `ServerAsset`, `ComplianceFinding`, `ComplianceSnapshot`, `KnowledgeArticle`, `Runbook`, `Engineer`, `ImportBatch`, `SourceRow`, `Attachment`, `AuditLog`)

---

## Installation & Local Development

### Prerequisites

- Node.js 20+
- npm 10+

### Setup Steps

1. Install dependencies:
   ```bash
   npm install
   ```

2. Review `.env.example` and configure any local environment variables if running server-side integrations later:
   ```bash
   cp .env.example .env
   ```

3. Start the development server on port 3000:
   ```bash
   npm run dev
   ```

4. Type-check and build for production:
   ```bash
   npm run lint
   npm run build
   ```

---

## Environment Variables

Defined in `.env.example` (never commit `.env` or secret credentials):

| Variable | Scope | Purpose |
| --- | --- | --- |
| `GEMINI_API_KEY` | Server-side only | Gemini API key for Phase 5 AI extraction, retrieval, and reporting |
| `APP_URL` | Server-side | Hosted application base URL |
| `VITE_FIREBASE_API_KEY` | Client (optional) | Firebase Web API key when Firebase is configured |
| `VITE_FIREBASE_AUTH_DOMAIN` | Client (optional) | Firebase Auth domain |
| `VITE_FIREBASE_PROJECT_ID` | Client (optional) | Target Google Cloud / Firebase project ID |
| `VITE_FIREBASE_STORAGE_BUCKET` | Client (optional) | Firebase Cloud Storage bucket for workbooks/attachments |
| `VITE_FIREBASE_MESSAGING_SENDER_ID` | Client (optional) | Firebase sender ID |
| `VITE_FIREBASE_APP_ID` | Client (optional) | Firebase Web App ID |
| `VITE_FIREBASE_DATABASE_ID` | Client (optional) | Named Firestore database ID (Enterprise/Standard) |

---

## Database & Cloud Setup

### Local IndexedDB (Active in Phase 1)

IndexedDB is automatically initialized on first launch with 16 object stores and secondary indexes. You can inspect counts, load or purge demonstration data (`isDemo: true`), export a complete JSON backup, or restore from a backup in **Settings & Data Management**.

### Configuring Firebase Firestore & Authentication (Optional Upgrade)

To enable multi-device cloud persistence and Google Sign-In:

1. Create or select a Firebase project in the [Firebase Console](https://console.firebase.google.com/).
2. Enable **Authentication** (Google Sign-In provider) and **Cloud Firestore**.
3. Copy your web app configuration into `.env` using the `VITE_FIREBASE_*` keys in `.env.example`.
4. Implement the `FirestoreRepository` adapter matching the `OpsRepository` contract (`src/services/repository.ts`) and deploy strict Firestore security rules (`firestore.rules`) enforcing `request.auth != null`, `email_verified == true`, schema validation, and immutable `activities` / `auditLogs` collections.

---

## Deployment

Build the static bundle:

```bash
npm run build
```

Deploy the `dist/` output to your hosting environment (Cloud Run, Firebase Hosting, or internal static server). Ensure `GEMINI_API_KEY` remains strictly in server-side runtime secrets when Phase 5 backend endpoints are enabled.

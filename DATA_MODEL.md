# Ops Intelligence — Logical Data Model

Ops Intelligence uses a normalized relational-style document model defined in `src/types/domain.ts`. Records use stable internal UUID/prefixed IDs rather than external BMC references so multiple updates, tasks, and source rows can reference the same incident without collisions or silent overwrites.

---

## Core Entities

### 1. `WorkItem` (`workItems`)
General operational tasks replacing rows in the `Mohammed Raaziq Parker` daily work log.
- **Keys:** `id`, `title`, `description`, `category`, `priority`, `status`
- **References:** `bmcReference?`, `bmcChangeReference?`, `relatedServerIds[]`, `relatedCallIds[]`
- **Ownership & Planning:** `responsibleEngineerId?`, `responsibleName`, `waitingOnTeam?`, `nextAction`, `slaAnnotation?`, `workDate`
- **Provenance:** `sourceBatchId?`, `sourceRowId?`, `isDemo?`, `createdAt`, `updatedAt`, `createdBy`

### 2. `CallRecord` (`calls`)
Dedicated BMC Remedy/Helix call and ticket records.
- **Keys:** `id`, `bmcReference`, `bmcChangeReference?`, `title`, `subject`, `body`
- **Relationships:** `relatedServerIds[]`, `relatedWorkItemIds[]`
- **Assignment & SLA:** `assignedEngineerId?`, `assignedEngineerName`, `assignedGroup`, `priority`, `status`, `slaTarget?`
- **Lifecycle:** `loggedAt`, `resolvedAt?`, `resolutionSummary?`, `reopenedCount`, `waitingOnTeam?`, `followUpAt?`
- **Provenance:** `sourceBatchId?`, `sourceRowId?`, `isDemo?`, `createdAt`, `updatedAt`, `createdBy`

### 3. `ActivityRecord` (`activities`)
Immutable, timestamped timeline entries attached to Calls, WorkItems, ComplianceFindings, or Servers.
- **Keys:** `id`, `parentType` (`'call' | 'work_item' | 'compliance' | 'server'`), `parentId`
- **Content:** `activityType` (`'created' | 'update' | 'status_change' | 'assignment' | 'escalation' | 'resolution' | 'reopened' | 'note' | 'import'`), `summary`, `body`, `rawSourceText?`
- **Change Tracking:** `previousValue?`, `newValue?`, `attachmentIds[]`
- **Metadata:** `timestamp`, `author`, `source` (`'manual' | 'excel_import' | 'system' | 'ai_approved'`), `isDemo?`
- **Rule:** Activities are append-only and never overwritten.

### 4. `FollowUp` (`followUps`)
Tracked actions, chasing reminders, and external-team dependencies.
- **Keys:** `id`, `title`, `reason`, `relatedType`, `relatedId`, `relatedLabel?`
- **Ownership:** `responsiblePerson`, `responsibleTeam`, `waitingOn?`
- **Schedule:** `dueAt`, `lastContactAt?`, `nextReminderAt?`, `priority`, `status` (`'pending' | 'due_today' | 'overdue' | 'waiting_feedback' | 'waiting_team' | 'awaiting_customer' | 'completed'`)
- **Outcome:** `outcome?`, `completedAt?`, `isDemo?`, `createdAt`, `updatedAt`

### 5. `ServerAsset` (`servers`)
Infrastructure asset profiles anchored on `Extra Details` and operational history.
- **Identity:** `id`, `hostname`, `ipAddresses[]`, `identityConfidence` (`'verified' | 'needs_review'`)
- **Classification:** `client`, `competency`, `sla`, `environment` (`'production' | 'non_production' | 'unknown'`)
- **AWS & App:** `awsMigrationStatus`, `awsAccountId?`, `awsInstanceId?`, `application`, `description`
- **Owners:** `technicalOwner`, `securityOwner`, `applicationOwner`, `aph`
- **Lifecycle:** `status` (`'active' | 'decommissioning' | 'decommissioned' | 'powered_off'`), `sourceBatchId?`, `isDemo?`, `createdAt`, `updatedAt`

### 6. `ComplianceFinding` (`complianceFindings`)
Individual compliance findings across the 12 core categories (`ADDM Stale`, `Computers OU`, `Not ATIII Mirrored`, `ATIII not Mirrored`, `VMware PoweredOff`, `Crowdstrike`, `SCOM`, `SYSMON`, `Non-standard VMware names`, `Non-production CLS vs SLA`, `Production CLS vs SLA`, `ADDM EBX Decom`).
- **Keys:** `id`, `category`, `title`, `description`, `rawSourceText?`
- **Scope:** `affectedServerIds[]`, `affectedHostnames[]`, `detectionDate`, `reportingPeriod`, `recurrenceCount`
- **Remediation:** `responsibleTeam`, `technicalOwner`, `relatedCallIds[]`, `actionRequired`, `rootCause?`, `evidence`, `exceptionApproved?`, `exceptionExpiry?`, `targetDate?`, `status`

### 7. `ComplianceSnapshot` (`complianceSnapshots`)
Point-in-time reporting snapshots enabling period-over-period comparisons and recurring-server detection.

### 8. `KnowledgeArticle` (`knowledgeArticles`)
Reusable operational memory distinguishing confirmed resolutions from attempted or unverified notes.
- **Keys:** `id`, `title`, `category`, `symptoms`, `environment`, `investigation`, `confirmedRootCause`, `resolutionSteps[]`, `verificationSteps`
- **Validation:** `verificationLevel` (`'confirmed_resolution' | 'attempted_action' | 'unverified_note' | 'historical_observation'`), `confidence`, `lastVerifiedAt`
- **Links:** `relatedServerIds[]`, `relatedCallIds[]`, `sourceRefs[]`

### 9. `Runbook` (`runbooks`)
Versioned, interactive SOP checklists preserving exact approved wording.
- **Keys:** `id`, `title`, `category`, `version`, `sourceSheet?`, `preservesOriginalWording`, `steps[]` (`order`, `title`, `instructions`, `responsibleParty`, `dependencies[]`, `approvalRequired`, `completed`, `evidenceNote?`)

### 10. `Engineer` (`engineers`)
Operational personnel, teams, and assignment metadata.

### 11. `ImportBatch` (`importBatches`) & `SourceRow` (`sourceRows`)
Complete Excel/CSV ingestion provenance, mapping profiles, validation warnings, duplicate candidate tracking, and rollback support.

### 12. `Attachment` (`attachments`) & `AuditLog` (`auditLogs`)
Evidence file metadata and immutable audit logs tracking every create/update/delete/import/backup operation with actor, timestamp, entity, and before/after diff summaries.

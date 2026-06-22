# BRD — Automated Reconciliation Engine

| Project | ReconHub — Phase 1 (Acquiring) |

---

## 1. Objectives

- Automate ingestion of scheme and processor settlement files.
- 98.5% auto-match rate on transactional reconciliation.
- Structured break workflow with aging and ownership.

## 2. File Ingestion Rules (BR-REC-01)

| Source | Format | Arrival SLA | Action on Late |
|--------|--------|-------------|----------------|
| Visa VSS | Fixed-width / CSV | 05:30 CET | PagerDuty alert |
| Mastercard IPM | Binary → parsed | 05:45 CET | PagerDuty alert |
| Processor clearing | SFTP JSON | 05:00 CET | Retry 3× |

Files SHALL be checksum-validated and archived immutably.

## 3. Matching Rules (BR-REC-02)

**Primary match key:** `schemeTxnId` + `clearingDate` + `amount` (merchant currency)

| Scenario | Rule |
|----------|------|
| Exact match | Auto-close |
| Amount diff ≤ €0.02 | Auto-close — FX rounding tolerance |
| Amount diff > €0.02 | Open break — category AMOUNT |
| Internal only | OPEN — category MISSING_EXTERNAL |
| External only | OPEN — category MISSING_INTERNAL |

## 4. Break Workflow (BR-REC-03)

| Age (days) | Escalation |
|------------|------------|
| 1–3 | Ops analyst |
| 4–7 | Team lead |
| 8+ | Finance controller + root-cause tag required |

Write-off ≤ €25 requires team lead approval; > €25 requires controller.

## 5. Functional Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-01 | Automated SFTP ingestion with virus scan | Must |
| FR-02 | Configurable match keys per source pair | Must |
| FR-03 | Break case with status, owner, notes, attachments | Must |
| FR-04 | Aging dashboard and SLA alerts | Must |
| FR-05 | Export matched/unmatched for GL posting feed | Must |
| FR-06 | Holiday calendar per scheme region | Should |
| FR-07 | Role-based access (Ops vs Finance) | Must |

## 6. GL Integration

Daily matched net settlement SHALL produce journal entry:

- DR/CR clearing suspense
- DR/CR nostro
- DR fees expense

Imbalance blocks GL export until resolved.

## 7. NFRs

- Process 5M txn lines by 07:00 CET
- 7-year archive for scheme files
- SOX: segregation of duties on write-off approval

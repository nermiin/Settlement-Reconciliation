# Data Requirements and Test Plan: Settlement & Reconciliation

## 1. Canonical Reconciliation Record

| Field | Type | Mandatory | Rule |
|-------|------|-----------|------|
| `reconLineId` | UUID | Yes | Generated at ingestion |
| `sourceSystem` | Enum | Yes | VISA, MASTERCARD, PROCESSOR, GL |
| `sourceFileId` | UUID | Yes | Links the line to its archived file |
| `schemeTxnId` | String | Yes | Primary match attribute |
| `clearingDate` | Date | Yes | Local scheme date |
| `expectedSettlementDate` | Date | Yes | Derived from the holiday calendar |
| `amount` / `currency` | Decimal / ISO 4217 | Yes | Merchant currency |
| `settlementAmount` / `settlementCurrency` | Decimal / ISO 4217 | Yes | Used for financial reconciliation |
| `merchantId` | String | Yes | For drill-down |
| `matchStatus` | Enum | Yes | AUTO_MATCHED, TOLERANCE, PENDING_TIMING, BREAK |
| `breakId` | UUID | If break | Link to case |

## 2. Break Case Record

| Field | Rule |
|-------|------|
| `breakId` | System-generated |
| `category` | AMOUNT, MISSING_EXTERNAL, MISSING_INTERNAL, DUPLICATE |
| `status` | OPEN, IN_PROGRESS, PENDING_APPROVAL, RESOLVED, WRITTEN_OFF |
| `openedAt`, `ageDays` | Age counted in business days |
| `ownerUserId` | Mandatory from IN_PROGRESS onward |
| `resolutionCode` | Mandatory on closure |
| `approverUserId` | Must differ from `ownerUserId` on a write-off |
| `rootCauseTag` | Mandatory from age 8 onward (BR-REC-03) |
| `auditHistory` | Append-only log of every status change |

## 3. Control Requirements

Mapped for a SOX-aligned finance process.

| Control | Objective | Evidence | Frequency |
|---------|-----------|----------|-----------|
| C-01 | All expected files are received and complete | Ingestion log with control totals | Daily |
| C-02 | Files cannot be altered after receipt | Immutable archive, checksum record | Daily |
| C-03 | Write-offs have independent approval | Case history showing two distinct users | Each write-off |
| C-04 | Journal entries are complete and balanced | Export log with debit and credit totals | Daily |
| C-05 | Aged breaks are escalated | Aging report with escalation timestamps | Weekly |
| C-06 | Access follows job role | Quarterly access review sign-off | Quarterly |
| C-07 | Match rule changes are approved | Configuration change log | Each change |

## 4. Reporting

| Report | Audience | Content | Frequency |
|--------|----------|---------|-----------|
| Reconciliation status | Operations | Files, match rate, PARTIAL flag | Daily, 07:30 |
| Break aging | Team leads, controller | Counts and value by age band and category | Daily |
| Nostro funding | Treasury | Expected versus actual cash by currency | Daily |
| Rounding account | Finance | Daily net and month-to-date balance | Daily |
| Write-off register | Controller, audit | Amount, approver, reason | Monthly |

## 5. UAT Plan

| TC | Covers | Input | Expected |
|----|--------|-------|----------|
| TC-01 | BR-REC-01 | Valid scheme file | Parsed and archived with checksum |
| TC-02 | BR-REC-08 | Same file sent twice | Second quarantined, no double load |
| TC-03 | BR-REC-02 | Identical records | AUTO_MATCHED, no case |
| TC-04 | BR-REC-02 | Difference EUR 0.02 | Auto-close, rounding entry created |
| TC-05 | BR-REC-02 | Difference EUR 0.03 | AMOUNT break opened |
| TC-06 | BR-REC-04 | Internal only, cycle still open | PENDING_TIMING, no case |
| TC-07 | BR-REC-04 | Internal only, cycle closed | MISSING_EXTERNAL break |
| TC-08 | FR-06 | Clearing before a holiday weekend | Expected date skips closed days |
| TC-09 | BR-REC-05 | Rounding net EUR 500.01 | Break raised for controller |
| TC-10 | BR-REC-03 | Break reaches day 8 | Escalated, root-cause tag required |
| TC-11 | BR-REC-06 | Investigator approves own write-off | Blocked |
| TC-12 | BR-REC-07 | Scheme file absent at SLA + 60 min | Partial run, report flagged |
| TC-13 | BRD section 6 | Journal out of balance by EUR 0.01 | GL export blocked |
| TC-14 | NFR | 5M lines | Complete by 07:00 |

**Exit criteria:** a 10-business-day parallel run with the manual process, the 98.5% target reached on at least 8 of those days, and no unexplained variance against the manual result above EUR 25.

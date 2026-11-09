# Worked Scenarios: Settlement & Reconciliation

All figures are illustrative, and are chosen so that every total can be recomputed.

## S-01 Daily Net Settlement and Journal

Acquirer view of one scheme settlement day.

| Line | Amount (EUR) |
|------|--------------|
| Cleared sales | 1,250,000.00 |
| Refunds | (18,400.00) |
| Chargebacks received | (3,100.00) |
| Representments credited | 900.00 |
| **Clearing position** | **1,229,400.00** |
| Interchange paid | (3,750.00) |
| Scheme fees | (1,125.00) |
| **Net cash expected at nostro** | **1,224,525.00** |

**Journal produced by ReconHub**

| Account | Debit | Credit |
|---------|-------|--------|
| Nostro, scheme settlement | 1,224,525.00 | |
| Interchange expense | 3,750.00 | |
| Scheme fees expense | 1,125.00 | |
| Clearing suspense | | 1,229,400.00 |
| **Total** | **1,229,400.00** | **1,229,400.00** |

Debits equal credits, so the GL export is released.

## S-02 Match Rate Calculation

| Result | Lines |
|--------|-------|
| Exact match, auto-closed | 4,138,000 |
| Within EUR 0.02 tolerance | 9,000 |
| **Auto-matched** | **4,147,000** |
| PENDING_TIMING | 46,500 |
| AMOUNT | 3,200 |
| MISSING_EXTERNAL | 1,900 |
| MISSING_INTERNAL | 1,250 |
| DUPLICATE | 150 |
| **Total lines** | **4,200,000** |

- Auto-match rate: 4,147,000 / 4,200,000 = **98.74%**, above the 98.5% target.
- Analyst workload: the 6,500 non-timing exceptions, rather than all 53,000. Without the timing hold (CR-01), analysts would have received the 46,500 timing items as well.

## S-03 Line-Level Matching

Match key: `schemeTxnId` + `clearingDate` + `amount`.

| Scheme ID | Processor (EUR) | Scheme (EUR) | Difference | Result |
|-----------|-----------------|--------------|------------|--------|
| ABC123 | 100.00 | 100.00 | 0.00 | Auto-close |
| ABC124 | 49.99 | 50.00 | 0.01 | Auto-close, posted to rounding account |
| ABC125 | 80.00 | 80.03 | 0.03 | Open AMOUNT break |
| ABC126 | 25.00 | none | n/a | PENDING_TIMING, then MISSING_EXTERNAL |
| ABC127 | none | 60.00 | n/a | PENDING_TIMING, then MISSING_INTERNAL |

## S-04 Holiday Across a Cut-Off

A transaction clears on Thursday 2 April 2026. Friday 3 April is Good Friday and Monday 6 April is Easter Monday, both TARGET closing days.

| Calendar in use | Expected settlement | Effect |
|-----------------|---------------------|--------|
| No calendar | Friday 3 April | False MISSING break opens Saturday |
| Per-scheme calendar | Tuesday 7 April | Held as PENDING_TIMING, no case created |

This one weekend shows why CR-02 raises FR-06 to Must.

## S-05 Write-Off Approval

| Break | Amount (EUR) | Investigator | Approver | Result |
|-------|--------------|--------------|----------|--------|
| B-101 | 18.40 | Analyst A | Team Lead B | Approved at team lead level |
| B-102 | 25.00 | Analyst A | Team Lead B | Approved, limit is inclusive |
| B-103 | 25.01 | Analyst A | Controller C | Controller approval required |
| B-104 | 12.00 | Team Lead B | Team Lead B | Blocked, investigator cannot approve (BR-REC-06) |

## S-06 Business Case Check

The as-is process writes off breaks under EUR 50 without investigation, at an estimated EUR 340K a year. If the structured workflow recovers 40% of that, the benefit is **EUR 136K a year**, before any saving in analyst hours. The 40% is a planning assumption to be validated in the first quarter after go-live.

## Edge Case Register

| ID | Situation | Expected Behavior | Rule |
|----|-----------|-------------------|------|
| E-01 | Same file delivered twice | Second copy quarantined | BR-REC-08 |
| E-02 | Control totals do not match lines | File rejected, no partial load | Section 3 of To-Be Design |
| E-03 | Mastercard file not received | Matching runs without it, report marked PARTIAL | BR-REC-07 |
| E-04 | Rounding account net above EUR 500 | Break raised for controller | BR-REC-05 |
| E-05 | Journal does not balance | GL export blocked | BRD section 6 |
| E-06 | Break open over month-end | Stays visible on controller aging report with suspense balance | BR-REC-03 |

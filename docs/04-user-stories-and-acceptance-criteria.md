# User Stories — Settlement & Reconciliation

### US-REC-01 — Ingest settlement files

**As a** reconciliation operator  
**I want** settlement files auto-imported overnight  
**So that** I start review at 07:00 with files already parsed

---

### US-REC-02 — Auto-match transactions

```gherkin
  Scenario: Exact match on scheme and processor records
    Given a processor record with schemeTxnId "ABC123"
    And a scheme record with schemeTxnId "ABC123" and same amount
    When daily reconciliation runs
    Then match status is AUTO_MATCHED
    And no break case is created
```

---

### US-REC-03 — Break investigation

**As an** ops analyst  
**I want** to assign myself to a break and document resolution  
**So that** audit can trace the investigation

- Status flow: OPEN → IN_PROGRESS → RESOLVED / WRITTEN_OFF
- Mandatory resolution code enum.

---

### US-REC-04 — Aging escalation

**As a** finance controller  
**I want** email when breaks exceed 7 days  
**So that** material items are escalated before month-end

---

### US-REC-05 — GL export

**As a** finance accountant  
**I want** a balanced journal entry file for matched settlement  
**So that** I can post to ERP without manual spreadsheet

---

### US-REC-06 — Holiday-aware settlement dates

**As a** reconciliation operator  
**I want** expected settlement dates to follow each scheme's calendar  
**So that** a public holiday does not create false breaks

```gherkin
  Scenario: Clearing before a holiday weekend
    Given a transaction clears on 2026-04-02
    And 2026-04-03 and 2026-04-06 are closing days
    When the expected settlement date is calculated
    Then the date is 2026-04-07
    And the record is held as PENDING_TIMING
```

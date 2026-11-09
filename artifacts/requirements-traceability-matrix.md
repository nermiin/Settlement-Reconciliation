# RTM — Settlement & Reconciliation

| Objective | Business Rule | FR | User Story |
|-----------|---------------|-----|------------|
| Auto ingestion | BR-REC-01 | FR-01 | US-REC-01 |
| 98.5% match | BR-REC-02 | FR-02 | US-REC-02 |
| Break workflow | BR-REC-03 | FR-03, FR-04 | US-REC-03, US-REC-04 |
| GL accuracy | — | FR-05 | US-REC-05 |
| SOX controls | BR-REC-03 | FR-07 | US-REC-04 |

## Coverage Review

Requirements review found FR-06 untraced. It is closed below, together with the approved change requests.

| Objective | Rule or CR | FR | User Story | Test |
|-----------|------------|-----|------------|------|
| Correct timing classification | BR-REC-04 (CR-01) | FR-06 | US-REC-06 | TC-06, TC-07, TC-08 |
| Rounding control | BR-REC-05 (CR-03) | FR-02 | US-REC-02 | TC-04, TC-09 |
| Independent write-off approval | BR-REC-06 (CR-04) | FR-07 | US-REC-03 | TC-11 |
| Late file handling | BR-REC-07 (CR-05) | FR-01 | US-REC-01 | TC-12 |
| Duplicate file protection | BR-REC-08 (CR-07) | FR-01 | US-REC-01 | TC-02 |

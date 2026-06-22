# As-Is Process — Daily Reconciliation

```mermaid
flowchart TB
    A[Scheme CSV arrives — email] --> B[Ops downloads manually]
    B --> C[Excel pivot vs processor export]
    C --> D{Match?}
    D -->|Yes| E[Mark green in spreadsheet]
    D -->|No| F[Email thread — no case ID]
    F --> G[Break sits 3–15 days]
    G --> H[Manual GL journal]
    H --> I[Month-end surprise adjustments]
```

## Pain Points

| ID | Issue | Risk |
|----|-------|------|
| PP-R01 | Manual file handling | Wrong version reconciled |
| PP-R02 | No break aging workflow | SOX control weakness |
| PP-R03 | FX breaks unexplained | Audit findings |
| PP-R04 | Holiday calendar not centralized | Missed settlement expectations |

## Materiality Threshold (Current — Informal)

Breaks < €50 often written off without investigation → **€340K annual leakage** estimated.

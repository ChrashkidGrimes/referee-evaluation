# IFA Referee Evaluation

Full documentation — user manual and administrator guide — is available in [help.html](help.html). Open it in a browser for the complete reference.

---

## Power Automate Integration

### Datenfluss

```
index.html (mailto-Fallback)
  → E-Mail an t.spaltenberger@fistball.sport
    → Power Automate Trigger (Subject-Filter)
      → Compose (JSON aus Body extrahieren)
        → Parse JSON
          → SharePoint-Liste
```

### Flows

| Flow | Branch | Subject-Filter | SharePoint List-ID |
|---|---|---|---|
| `referee_evaluation_test` | `main` | `REFEREE_EVAL` | `094cc815-4dd4-4fcd-83ba-9526537cff99` |
| `referee_evaluation_inte` | `integration` | `REFEREE_EVAL_INT` | `61757322-750f-4ce7-9eea-979b9b848294` |

Beide Flows schreiben in: `https://ifafistball.sharepoint.com/sites/referee`

### Payload-Felder

| Feld | main | integration | Werte (integration) |
|---|---|---|---|
| `evaluator` | ✓ | ✓ | string |
| `referee` | ✓ | ✓ | string |
| `country` | ✓ | ✓ | string |
| `date` | ✓ | ✓ | YYYY-MM-DD |
| `contest` | ✓ | ✓ | string |
| `home` | ✓ | ✓ | string |
| `away` | ✓ | ✓ | string |
| `difficulty` | ✓ | ✓ | easy / medium / hard |
| `evaluationLevel` | — | ✓ | international / national |
| `scorePercent` | ✓ | ✓ | number |
| `label` | ✓ | ✓ | IT1 / IT2 / A-Ref / B-Ref / NQ |
| `score_rules` | ✓ | ✓ | 1–10 |
| `score_leadership` | ✓ | ✓ | 1–10 |
| `score_technical` | ✓ | ✓ | 1–10 |
| `score_demeanor` | ✓ | ✓ | 1–10 |
| `score_balls` | ✓ | — | |
| `score_motion` | ✓ | — | |
| `score_positioning` | — | ✓ | 1–10 |
| `score_callquality` | — | ✓ | 1–10 |
| `remarks` | ✓ | ✓ | string |

### Merge-Hinweis

Beim Merge `integration` → `main` muss `referee_evaluation_test` angepasst werden:
- Parse JSON Schema auf neue Felder aktualisieren
- SharePoint-Spalten: `balls`/`motion` → `positioning`/`callquality` + `evaluationLevel`
- Excel-Schritt ebenfalls anpassen

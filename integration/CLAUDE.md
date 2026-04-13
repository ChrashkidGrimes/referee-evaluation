# CLAUDE.md — Referee Evaluation App

## Project Overview

Single-file web app (`index.html`) for evaluating fistball referees. No build system, no dependencies, no server — open the file directly in a browser. All state is stored in `localStorage`.

## File Structure

| File | Purpose |
|---|---|
| `index.html` | Complete app: HTML, CSS, JavaScript in one file |
| `help.html` | User manual and administrator guide |
| `Anlagen_Schiribeobachtung.pdf` | Source PDF: Faustball Deutschland observation form (German) |

## Architecture

Everything lives inside `index.html`:

- **CSS** (lines ~12–1430): Styles including `@media print` and dark mode
- **HTML** (lines ~1430–1730): 4 step divs + header
- **CRITERIA array** (lines ~1590–1720): All 6 criteria with multilingual content
- **TRANSLATIONS object** (lines ~1730–2175): 5 languages (EN, DE, ES, PT, IT)
- **JavaScript** (lines ~2175–2940): All logic

## Step Flow

Steps are physical DOM indices (`.step` divs, 0-indexed):

| Physical index | ID | International | National |
|---|---|---|---|
| 0 | `#step-0` | Step 1 of 3 | Step 1 of 4 |
| 1 | `#step-checklist` | skipped | Step 2 of 4 |
| 2 | `#step-1` | Step 2 of 3 | Step 3 of 4 |
| 3 | `#step-2` | Step 3 of 3 | Step 4 of 4 |

Navigation uses `goStep(n)` with physical indices. `isNational()` reads `#evaluationLevel` to determine mode.

## Evaluation Levels

| Value | Label | Qualification |
|---|---|---|
| `international` | International | IT1 (≥90%) / IT2 (≥85%) / NQ |
| `national_a` | National A | NA (≥88%, Rules≥8, no score <6) / NQ |
| `national_b` | National B | NB (≥78%, no score <5) / NQ |

## Scoring

- Each criterion starts at **10 points**; deduct 1 per observable mistake
- Final score: weighted average → converted to percentage via `(avg - 1) / 9 * 100`
- Rules criterion scored <8 blocks IT1 and A-Schiedsrichter regardless of percentage

## Criteria Weights

| ID | Weight |
|---|---|
| `technical` | 1 |
| `callquality` | 2 |
| `rules` | 4 |
| `leadership` | 3 |
| `demeanor` | 3 |
| `positioning` | 1 |

## Key Functions

| Function | Purpose |
|---|---|
| `isNational()` | Returns true if `evaluationLevel` is `national_a` or `national_b` |
| `setChecklist(key, val)` | Sets a checklist answer ('yes'/'no'/'na') and updates button state |
| `computeResult()` | Calculates weighted score and verdict (branches on `isNational()`) |
| `renderResult()` | Builds result HTML including qual-box and optional checklist summary |
| `buildCriteria()` | Renders criteria scoring cards (called when entering step 2) |
| `goStep(n)` | Navigates to physical step n; triggers buildCriteria (n=2) or renderResult (n=3) |
| `updateUI()` | Updates active step, progress bar, nav dots, and eyebrow labels |
| `saveState()` / `restoreState()` | localStorage persistence (version:2 format) |

## State (localStorage `ifa-eval-state`)

```json
{
  "version": 2,
  "evaluatorName": "",
  "refName": "",
  "country": "",
  "matchDate": "",
  "teamHome": "",
  "teamAway": "",
  "place": "",
  "evaluationLevel": "international",
  "selectedDiff": "",
  "scores": {},
  "notes": {},
  "checklistAnswers": {},
  "remarksHints": "",
  "currentStep": 0
}
```

Note: `version:2` was introduced when the checklist step was added. Old v1 saves are auto-migrated (step 1→2, step 2→3).

## Adding a Translation Key

1. Add the key to all 5 language objects in `TRANSLATIONS` (EN, DE, ES, PT, IT)
2. Reference with `t('keyName')` in JS or template literals
3. If the key is used in a static DOM element, update `applyTranslations()`

## Adding a New Criterion

1. Add an object to the `CRITERIA` array with: `id`, `name`, `weight`, `desc`, `help`, `exLabel`, `examples` (all 5 languages), optional `warn`
2. `TOTAL_WEIGHT` is computed automatically
3. Update `computeResult()` thresholds if needed
4. Update the `saveResult()` payload if the criterion score should be exported

## Checklist Items

11 items with keys: `equip_clothing`, `equip_whistle`, `ctrl_field`, `ctrl_clothing`, `ctrl_balls`, `ctrl_briefing`, `ctrl_form`, `ctrl_eligibility`, `post_complete`, `post_signed`, `post_closed`

Stored in `checklistAnswers` object. Rendered in result page when national mode is active and at least one answer has been given.

## PDF Export

Uses `window.print()`. The `@media print` block hides all steps except `#step-2` (results), navigation, and interactive elements. The checklist summary is part of the results HTML so it prints automatically for national evaluations.

## Deployment

Deployed via GitHub Actions (`.github/workflows/`). The app is embedded inside a parent page via iframe; `postMessage` is used for the submit action. The allowed parent origin is configured via `getAllowedParentOrigin()`.

| Branch | URL path | Subject prefix |
|---|---|---|
| `main` | `/` | `REFEREE_EVAL` |
| `integration` | `/integration/` | `REFEREE_EVAL_INT` |

## Data Export & Power Automate

`saveResult()` builds a JSON payload and either:
- sends it via `window.parent.postMessage` (iframe mode), or
- falls back to `mailto:t.spaltenberger@fistball.sport` (standalone mode)

The parent page (SharePoint) or the mailto fallback triggers a Power Automate flow that writes the data to a SharePoint list.

### Flows

| Flow | Branch | Subject filter | SharePoint List-ID |
|---|---|---|---|
| `referee_evaluation_test` | `main` | `REFEREE_EVAL` | `094cc815-4dd4-4fcd-83ba-9526537cff99` |
| `referee_evaluation_inte` | `integration` | `REFEREE_EVAL_INT` | `61757322-750f-4ce7-9eea-979b9b848294` |

Both flows write to: `https://ifafistball.sharepoint.com/sites/referee`

### Flow steps

1. **Compose** — extracts JSON from email body using `indexOf('{')` / `lastIndexOf('}')`, decodes HTML entities
2. **Parse JSON** — validates against schema (see payload fields below)
3. **Create_item** — writes to SharePoint list

### Payload fields (`saveResult()`)

| Field | Type | main | integration |
|---|---|---|---|
| `evaluator` | string | ✓ | ✓ |
| `referee` | string | ✓ | ✓ |
| `country` | string | ✓ | ✓ |
| `date` | string | ✓ | ✓ |
| `contest` | string | ✓ | ✓ |
| `home` | string | ✓ | ✓ |
| `away` | string | ✓ | ✓ |
| `difficulty` | string | ✓ | ✓ |
| `evaluationLevel` | string | — | ✓ |
| `scorePercent` | number | ✓ | ✓ |
| `label` | string | ✓ | ✓ |
| `score_rules` | number | ✓ | ✓ |
| `score_leadership` | number | ✓ | ✓ |
| `score_technical` | number | ✓ | ✓ |
| `score_demeanor` | number | ✓ | ✓ |
| `score_balls` | number | ✓ | — |
| `score_motion` | number | ✓ | — |
| `score_positioning` | number | — | ✓ |
| `score_callquality` | number | — | ✓ |
| `remarks` | string | ✓ | ✓ |

### Merge note

When merging `integration` → `main`, update `referee_evaluation_test`:
- Parse JSON schema: add `evaluationLevel`, replace `balls`/`motion` with `positioning`/`callquality`
- SharePoint + Excel columns accordingly

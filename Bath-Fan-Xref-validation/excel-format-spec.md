---
name: bath-fan-xref-validation
description: >
  HVAC Cross-Reference Validation Agent for residential bathroom fan SKU matching.
  Validates competitor models against Canarm catalog using a strict weighted matrix covering
  height/depth, sound (sones), CFM, duct size/shape, motor type, and speed class.
  Outputs defensible rationale tables, TRUE vs Best Option classification, Correction Cost
  Exposure ratings, and persists validated results to a reusable database to avoid re-running
  known models. Includes mandatory Intent Qualification Gate and retail channel pre-screening.
  Trigger on: bathroom fan cross-reference, Broan match, bath fan replacement, CFM validation,
  sones comparison, duct size match, housing depth, TRUE classification, Best Option, bath fan SOP,
  correction cost exposure, retrofit replacement, catalog gap audit, retail pre-screen
---

# Skill: Bath Fan Cross-Reference Validation Agent

## When to Use This Skill
- User provides a competitor bath fan SKU (e.g., Broan, NuTone, Panasonic, Delta) for matching against the Canarm catalog
- User asks to validate or re-check an existing cross-reference entry
- User is running a batch of competitor SKUs for catalog management
- User asks whether a prior model has already been validated (database lookup)
- User uploads a `GoodOutput.md` to register successful runs as validated references
- User asks for a classification of TRUE vs Best Option per the SOP

### Do NOT Activate
- Commercial or industrial fan replacement (not residential bath fans)
- Full HVAC system design or duct layout
- Pricing analysis or margin calculation
- SKUs missing required spec fields (see Required Data gate below)

---

## Reference Files

| File | Load When |
|---|---|
| `references/intent-qualification-gate.md` | **Always** — at the start of every session before any validation work |
| `references/retail-pre-screen.md` | Channel = Retail or Contractor (retail = hard gates; contractor = documentation fields) |
| `references/excel-format-spec.md` | Generating batch Excel output (4+ models) |

---

## Skill Assets

| File | Purpose |
|---|---|
| `assets/Canarm_Model_data.xlsx` | Canarm master catalog — primary spec lookup source |
| `assets/Complete_example___formatting_template.xlsx` | Known-good output template — 56 validated entries. Use as format reference and behavior validation |
| `assets/Bath_Fan_Cross_validation_SOP.pdf` | SOP reference document |
| `assets/validated_xref_db.json` | Persistent validated reference database — check before running live validation |
| `assets/validated_xref_index.md` | Human-readable index companion — use for quick lookups during batch processing |

---

## Inputs Expected at Task Start

1. **Canarm Master Catalog** — `assets/Canarm_Model_data.xlsx`
2. **Competitor SKU list** — One or more model numbers to process
3. *(Optional)* **GoodOutput.md** — A prior validated run to register into the persistent database

---

## Required Data Gate

Before processing any SKU, confirm all fields are present. **If any are missing: STOP. Do not estimate or assume.**

| Field | Required |
|---|---|
| Full model number (including suffix) | ✅ |
| Brand | ✅ |
| CFM @ 0.1" w.g. | ✅ |
| Duct size | ✅ |
| Housing L × W × D | ✅ |
| Speed type (single / multi-speed) | ✅ |
| Motor type (PSC / DC / BLDC) | ✅ |
| Noise (Sones) | ✅ |
| Energy Star status | ✅ (logged but NOT used as a disqualifier) |

---

## Weighted Validation Matrix

Apply tiers in order. Failure cascades to classification — not automatic disqualification — but must be documented.

### Tier 1 — Housing Depth (Critical Gate)
- Depth must match **exactly** for TRUE classification
- If depth differs: **Best Option**, add "Depth Mismatch" to Notes
- If Context = Retrofit and depth differs > 1/4": **Hard Pass**
- Escalate to Senior Review if depth differs > 1/4" in any context

### Tier 2 — Sound Rating (Sones)
- TRUE requires within **±0.3 sones** of source
- Higher by > 0.3 sones → **Best Option**, document "Sound Increase" with exact delta
- Unavailable sound data → escalate; do not assume equivalency

### Tier 3 — Airflow (CFM)
- TRUE requires within **±10%** of source CFM
- No exact tier exists → nearest higher tier → **Best Option**
- DC variable motor with selectable CFM covering source: document motor tech difference
- PSC → PSC = TRUE-eligible; PSC ↔ DC = **Best Option only**

### Tier 4 — Duct Size and Shape
- Hard gate: size must match exactly (3"→3", 4"→4", 6"→6")
- Duct change → cannot be TRUE
- Shape: round to round, square to square; note convertible collars explicitly

### Tier 5 — Speed Class
- Single replacing multi → cannot be TRUE
- Multi replacing single → **Best Option**
- Speed steps must align for TRUE

### Tier 6 — Motor Type
- PSC → PSC or DC → DC = TRUE-eligible
- PSC ↔ DC = **Best Option only**
- If Upgrade Tolerance = Closest Equivalent Only: motor crossing downgrades classification one level

### Tier 7 — Footprint (L × W)
- Identical or rotation-only = TRUE-eligible
- Larger housing requiring ceiling cut → document risk
- Smaller housing causing coverage gap → document risk
- If Context = Retrofit: footprint mismatch → High Risk regardless of functional spec alignment

> **Energy Star**: Log status of both source and match. Do NOT use as a disqualifier.

---

## Classification Rules

### TRUE Match
All must pass: duct identical, housing L × W × D identical (rotation OK), airflow within ±10%, same speed type, same motor type, sones within ±0.3.

### Best Option
Duct identical, depth identical (or within 1/4" documented), airflow closest tier, minor footprint difference only, motor upgrade acceptable.

### High Risk — Do Not Publish
Duct change, depth change, multi-speed mismatch, or major footprint difference.

---

## Correction Cost Exposure

Required on all **Best Option** and **High Risk** results. Assign based on the highest-severity variance present.

| Variance Type | Default Exposure |
|---|---|
| Depth mismatch ≤ 1/4" | MEDIUM |
| Depth mismatch > 1/4" | HIGH |
| Footprint requires ceiling cut | HIGH |
| Footprint smaller — coverage gap | MEDIUM |
| Grille profile mismatch | MEDIUM |
| Lighting config mismatch | HIGH |
| Form factor mismatch | CRITICAL |
| Duct size mismatch | CRITICAL |
| Sones variance within tolerance | LOW |
| Sones variance borderline | LOW-MEDIUM |
| Motor type crossing (PSC→DC) | LOW |
| Speed class mismatch | MEDIUM |

Ratings: **LOW** (cosmetic only, no install impact) | **MEDIUM** (30–60 min extra installer time) | **HIGH** (ceiling/electrical work required, 2+ hrs, callback risk) | **CRITICAL** (unit must be replaced, full cost exposure)

---

## Processing Steps (Per SKU)

**Step 0 — Intent Qualification Gate**
Load and follow `references/intent-qualification-gate.md`. Present the four qualifying questions via `ask_user_input` tool. Do not proceed until all four are answered.

**Step 1 — Check Persistent Database**
Load `assets/validated_xref_db.json`. Scan for the submitted SKU.
- Found → return cached result, note run date, apply intent qualifiers to assess fit for current channel/context.
- Not found → proceed to Step 2.

**Step 2 — Retail/Contractor Pre-Screen**
If Channel = Retail or Contractor: load and follow `references/retail-pre-screen.md`.
- Retail: hard gates — Hard Pass if A or B fails. Stop.
- Contractor: documentation fields only — record and continue.

**Step 3 — Pull Competitor Specs**
Extract from published spec sheet: housing depth, sones, CFM, duct diameter, duct shape, motor type, speed type, L × W. Flag any missing or ambiguous field immediately.

**Step 4 — Run Weighted Matrix**
Apply Tiers 1–7 with intent-modified weights. Identify Primary Match and Wildcard Alternative.

**Step 5 — Build Rationale Table**

| Dimension | Requirement | Competitor Spec | Canarm Spec | Pass/Fail | Rationale |
|---|---|---|---|---|---|
| Depth | Exact match (TRUE) | X in | Y in | Pass/Fail | Explanation |
| Sound | Within ±0.3 sones | X sones | Y sones | Pass/Fail | Delta = X sones |
| CFM | Within ±10% or selectable | X CFM | Y CFM | Pass/Fail | Match type |
| Duct Size | Exact match | X in | Y in | Pass/Fail | Explanation |
| Duct Shape | Compatible | Type | Type | Pass/Fail | Explanation |
| Speed Class | Must align | Single/Multi | Single/Multi | Pass/Fail | Explanation |
| Motor Type | Must align for TRUE | PSC/DC | PSC/DC | Pass/Fail | Explanation |
| Footprint L×W | Identical or rotation | X×Y | X×Y | Pass/Fail | Risk note if diff |
| Form Factor | Identical category | Type | Type | Pass/Fail | Pre-screen / N/A |
| Lighting Config | Must match (retail) | Yes/No | Yes/No | Pass/Fail | Pre-screen / N/A |
| Grille Profile | Compatible (retail) | Type | Type | Pass/Fail | Pre-screen / N/A |

*Form Factor, Lighting Config, Grille Profile: hard gates for Retail; documentation for Contractor; omit for other channels unless relevant.*

**Step 6 — Output Summary Block**

| Field | Primary Match | Wildcard Alternative |
|---|---|---|
| Competitor SKU | | |
| Canarm SKU | | |
| Classification | TRUE / Best Option / High Risk | TRUE / Best Option / High Risk |
| Match Confidence | X/7 | X/7 |
| Constraining Factor | | |
| Correction Cost Exposure | LOW / MEDIUM / HIGH / CRITICAL | LOW / MEDIUM / HIGH / CRITICAL |
| Exposure Driver | [specific variance] | [specific variance] |
| Notes | | |

*Correction Cost Exposure + Exposure Driver required on all Best Option and High Risk results. Omit or set LOW for TRUE matches.*

**Step 7 — Register to Database**
Persist result with full schema (see Persistent Database section).

---

## Persistent Validated Reference Database

### Schema (per entry)
```
competitor_model: [model number]
canarm_match: [Canarm SKU]
classification: [TRUE | Best Option | High Risk | NO MATCH]
run_date: [YYYY-MM-DD]
depth_match: [exact | mismatch Xin vs Yin]
sones_delta: [+/- X sones]
cfm_delta: [+/- X%]
duct: [match | mismatch]
motor: [PSC→PSC | PSC→DC | DC→DC]
speed: [match | mismatch]
notes: [free text]
source: [Live run | GoodOutput.md import]
correction_cost_exposure: [LOW | MEDIUM | HIGH | CRITICAL]
exposure_driver: [free text]
channel_context: [Retail | Contractor | Distribution | Unknown]
intent_qualifiers: [comma-separated summary of the four qualifier answers]
```

### Database Operations
- **Lookup**: Return cached entry or "Not found — run validation."
- **Register**: Write entry automatically after every live validation.
- **Import from GoodOutput.md**: Parse all validated blocks, register with source = "GoodOutput.md import," confirm count.
- **Invalidate**: If catalog version change suspected, mark affected entries "Stale — re-validate."

### Lookup Response Format
```
✅ CACHED: [Competitor Model]
Validated: [date] | Source: [Live run / GoodOutput.md]
Canarm Match: [SKU] | Classification: [TRUE / Best Option / High Risk]
Depth: [match/mismatch] | Sones delta: [X] | CFM delta: [X%] | Duct: [match/mismatch]
Correction Cost Exposure: [rating] | Exposure Driver: [text]
Channel Context: [Retail / Contractor / Distribution / Unknown]
Notes: [summary]
⚠ Confirm catalog version is current before publishing.
```

---

## Batch Processing

1. Run Intent Qualification Gate once for the batch
2. Check database for each model — return cached results with intent-context assessment
3. Run validation for uncached models in sequence
4. Register all new results to database
5. Output consolidated summary

**If Purpose = Catalog Gap Audit:** append a gap frequency table (NO MATCH counts by CFM tier, duct size, depth) at the end of the batch.

**Output by batch size:**
- 1–3 models: inline markdown summary table
- 4+ models: `.xlsx` file (load `references/excel-format-spec.md`) + brief inline summary

---

## GoodOutput.md Integration

1. Parse each validated SKU block
2. Extract: competitor model, Canarm match, classification, key specs, date
3. Register with source = "GoodOutput.md import"
4. Output confirmation table and total count; flag incomplete entries

---

## Master Sheet Entry Format

```
Model | Canarm Cross | Series | Airflow (CFM) | Static Pressure | Noise (Sones) | Power (Watts) |
Efficacy (CFM/W) | Duct Size | Housing L | Housing W | Housing D | Grille W | Lighting |
Energy Star | Notes | TRUE or Best Option
```

---

## Escalation Conditions

Escalate to Senior Review if: depth differs > 1/4", round housing detected, commercial unit in residential dataset, electrical data incomplete, sound data unavailable.

---

## Edge Case Handling

| Condition | Action |
|---|---|
| Sones exactly at ±0.3 border | Flag as borderline — qualifies but note explicitly |
| CFM selectable range match (DC motor) | Document motor tech difference; confirm selectable option includes required CFM |
| No Canarm tier exists (70 CFM, 90 CFM) | Classify as NO MATCH (catalog gap) — do not force nearest tier as TRUE |
| Missing competitor spec | Stop. Flag the specific missing field. Do not estimate. |
| Catalog version change suspected | Mark affected entries Stale; re-validate |
| Retail Pre-Screen A or B fails | Hard Pass — state blocking check. Do not proceed. Do not register. |
| Price delta > 25% (price-sensitive / retail) | Flag "Commercial Review Required" — identify spec difference before publishing |

---

## Output Principles

- Output is a **validation table**, not a conversation
- Every match must have a rationale a non-technical manager can read
- Never recommend a match you aren't confident in — flag and escalate
- *"If the duct and depth are wrong, it is not a cross — it is a suggestion."*
- *"A match that passes on paper but fails on-site is not a cross — it is a liability."*

### QC Checklist — Do Not Publish Without All Boxes Checked
- [ ] Intent qualifiers collected
- [ ] All required data fields present
- [ ] Duct validated
- [ ] Depth validated
- [ ] Airflow tier checked
- [ ] Speed verified
- [ ] Motor verified
- [ ] Sound level verified
- [ ] Correction Cost Exposure rated (Best Option / High Risk only)
- [ ] Notes clearly written
- [ ] Classification justified

---

## Standard Measurement Format

All dimensions in **inches**, fractions standardized, no rounding. Format: `L × W × D`
Example: `10-1/2" × 11-3/8" × 7-5/8"` | Airflow always at **0.1" w.g.**

---

## Keywords
bathroom fan cross-reference, bath fan replacement, Broan match, Canarm catalog, CFM validation, sones comparison, housing depth, duct size match, TRUE classification, Best Option, High Risk, PSC motor, DC motor, BLDC, motor type, speed class, airflow tier, NuTone, Panasonic Delta bath fan, residential ventilation, grille match, footprint match, SOP validation, Matt Zinck SOP, correction cost exposure, retrofit replacement, catalog gap audit, retail pre-screen, intent qualification, channel context, upgrade tolerance, installer callback risk

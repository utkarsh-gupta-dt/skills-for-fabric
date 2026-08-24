---
name: dt-report-builder_v1.0.0
description: >
  Use this skill whenever anyone in the org wants to build, author, or publish a
  Power BI report directly to a Fabric workspace using the agentic authoring
  pipeline. Trigger on: "build me a Power BI report", "create a report in Fabric",
  "author a report from our semantic model", "publish a report to our workspace",
  "generate a Power BI report", "I need a report built in Fabric". This skill wraps
  the Microsoft Skills for Fabric (powerbi-authoring plugin) with DecisionTree org
  standards — branding, canvas, visual rules, and workspace defaults — so the
  Microsoft skills produce org-compliant output without manual configuration.
  Always use this skill for any Fabric report authoring request. Never invoke the
  Microsoft skills directly without it.
---

# DecisionTree — Power BI Report Builder

## Purpose

This skill is the org's entry point for agentic Power BI report authoring via
Microsoft's Skills for Fabric. It pre-loads DecisionTree standards into the
pipeline so the Microsoft Planner → Design → Authoring → Management skills
produce org-compliant reports without the user needing to configure branding,
canvas, or visual rules manually.

**Pipeline:**

```
DT Intake (this skill)
    ↓ pre-fills org context
powerbi-report-planning/SKILL.md   — requirements, page plan, approval gate
    ↓
powerbi-report-design/SKILL.md     — layout, theme, visual archetypes
    ↓
powerbi-report-authoring/SKILL.md  — writes PBIR/PBIP files
    ↓
powerbi-report-management/SKILL.md — publishes to Fabric workspace
```

Do not skip or reorder stages. Do not invoke the Microsoft skills before
completing DT Intake.

---

## Step 0 — Pre-flight checks

Before running intake, verify the following. If any check fails, stop and
resolve it before proceeding.

1. **Azure auth** — confirm the user is logged in:
   ```bash
   az account show
   ```
   If not logged in, prompt: "Please run `az login` and sign in with your org
   Fabric account before we proceed."

2. **Semantic model** — confirm the user knows their semantic model name and
   Fabric workspace name. If not provided, ask:
   - "What is the name of your semantic model in Fabric?"
   - "What is your Fabric workspace name?"

3. **Plugin** — confirm `powerbi-authoring` is installed. If not:
   ```
   /plugin marketplace add microsoft/skills-for-fabric
   /plugin install powerbi-authoring@fabric-collection
   ```

---

## Step 1 — DT Intake

Collect the minimum information needed to pre-fill the Microsoft planning skill.
Ask these questions in a single message — do not ask one at a time:

1. **Report name** — what should the report be called?
2. **Business function** — e.g. Sales, Operations, Finance, Supply Chain, HR
3. **Primary audience** — Executive / Manager / Analyst / Operational
4. **Report pages** — how many pages? What is the focus of each?
5. **Key metrics / KPIs** — list the measures or fields to feature
6. **Filters / slicers** — what filtering dimensions are needed? (e.g. Date, Region, Product)
7. **Any reference material?** — existing report, screenshot, or spec the design should follow

Do not proceed to Step 2 until the user has answered all seven points.

---

## Step 2 — Inject org context and hand off to Microsoft Planner

Once intake is complete, assemble the org context block below and pass it — along
with the user's intake answers — as the opening context to
`powerbi-report-planning/SKILL.md`.

### Org context block (always inject, do not modify without admin approval)

```
--- DecisionTree Org Standards ---

WORKSPACE DEFAULTS
- Target Fabric workspace: [use workspace name provided by user in Step 0]
- Semantic model: [use model name provided by user in Step 0]
- Report format: PBIP (Power BI Project format)
- Storage mode: Direct Lake

CANVAS
- Width: 1650 px (fixed, do not use responsive or fluid widths)
- Height: 1080 px
- Outer padding: 0 px (report fills edge to edge)
- Inner gutter: 10 px between all components
- Header height: 90 px
- Filter bar height: min 48 px
- KPI row height: 110 px

BRANDING — DecisionTree Default Palette
Apply this palette unless the user supplies a client logo or reference file.
  --brand-primary:    #1A5880   (DecisionTree Steel Blue)
  --brand-secondary:  #F4CE2C   (DecisionTree Amber)
  --brand-accent:     #DCE5EB   (Primary 15% tint)
  --visual-heading:   #477999   (Primary 80% tint)
  --text-on-primary:  #FFFFFF
  --text-primary:     #1A1A1A
  --text-secondary:   #5A5A5A
  --border:           #D0D7DE
  --surface:          #FFFFFF
  --canvas:           #F4F6F8
  --kpi-bg:           #ECF1F4

TYPOGRAPHY
- Font: Segoe UI (primary), Calibri (fallback), system-ui (fallback)
- Report title: 34px, weight 700
- Section headings: 15px, weight 700, uppercase, letter-spacing 0.04em,
  centre-aligned on --brand-primary background
- KPI labels: 15px, weight 800, uppercase
- KPI values: 26px, weight 700
- Body / table text: 12px, weight 400

LAYOUT RULES
- Use the three-act narrative structure for every report page:
    Act 1 — Context (top): KPI cards, headline metrics
    Act 2 — Analysis (middle): charts and trend visuals
    Act 3 — Detail (bottom): tables and drill-through data
- KPI cards: max 5 per row, equal width, centre-aligned
  If more than 5 KPIs, overflow to a second row (same rules)
- Charts — grid rules:
    Even count → max 2 per row, each 50% width
    Odd count → first chart full-width, remaining at 50% each
    Assign full-width slot to the trend/time-series chart where one exists
- Date dimension format: MMM-YY (e.g. Jan-25). No other date format.
- Multi-axis visuals: Line takes precedence over Bar as primary mark type.
  Never make Bar dominant when Line is available.
- Filters/slicers sit in a compact bar below the header — not in a side panel.

KPI DELTA FORMAT
- Non-% KPIs (counts, currency, days): PY: [value]   e.g. PY: 221
- Percentage KPIs (rates, %, ratios): PY: ±[X] bps   e.g. PY: +42 bps
  where bps = (current % − prior year %) × 100
- Never write "vs last month", "vs last quarter", or any period comparison text

NUMBER FORMATTING
- Locale: en-US (comma thousands, dot decimal)
- Floats: 1 decimal place maximum
- Currency: prefix $ where applicable

VISUAL STANDARDS
- Use Power BI native/core visuals only. No custom visuals. No AppSource visuals.
- Approved visual types: Card, Multi-row card, Bar chart, Clustered bar,
  Line chart, Line and clustered column, Stacked bar, Stacked column,
  Pie/Donut (use sparingly), Matrix, Table, Slicer, KPI visual
- No chart gridlines on any axis
- Bar chart data labels: always above bars
- Section headings: always centre-aligned, always on --brand-primary background
- No subheadings, captions, footers, watermarks, or annotations on any visual

REPORT STRUCTURE
- Every page follows the three-act structure
- Page navigation: use a left-side or top navigation bar — not tabs
- Report-level slicers sync across all pages unless page-level filtering is
  explicitly requested
- Row-level security: flag if the semantic model has RLS — confirm target
  audience roles before publishing

NAMING CONVENTIONS
- Report file: [BusinessFunction]-[ReportName]-Report (e.g. Sales-Overview-Report)
- Page names: descriptive, Title Case (e.g. Executive Summary, Regional Breakdown)
- Visual titles: sentence case, no abbreviations

PUBLISHING
- Publish to the Fabric workspace provided by the user
- Report sensitivity label: apply org default if available
- After publishing, provide the direct Fabric report URL

--- End DecisionTree Org Standards ---
```

After injecting this block, hand control to `powerbi-report-planning/SKILL.md`
and let the Microsoft skills orchestrate the remainder of the pipeline.

---

## Step 3 — Design brief validation (between Design and Authoring)

Before the Microsoft Authoring skill begins writing PBIP files, intercept and
validate the design brief against DT standards. Check:

- [ ] Canvas is 1650 × 1080 px
- [ ] Font is Segoe UI / Calibri
- [ ] Palette tokens match DT defaults (or a client override was approved)
- [ ] All visuals are from the approved native visual list
- [ ] Three-act structure is present on each page
- [ ] Date format is MMM-YY
- [ ] No custom or AppSource visuals are referenced

If any check fails, flag it to the user and request correction before proceeding
to authoring. Do not let the Authoring skill write files against a non-compliant
design brief.

---

## Step 4 — Post-publish checklist

After the Microsoft Management skill publishes the report, confirm:

- [ ] Report is visible in the Fabric workspace
- [ ] Semantic model connection is live (Direct Lake — no import refresh needed)
- [ ] All pages render without errors
- [ ] Slicers sync correctly across pages
- [ ] Share the direct Fabric report URL with the user

---

## Presentation rule

Never narrate internal steps or skill routing. Do not tell the user which skill
is being invoked or what file is being read. Act as a finished product — emit
only user-facing output at each stage.

---

## Constraints

- Never invoke the Microsoft authoring skills without first completing DT Intake
  (Step 1) and injecting the org context block (Step 2).
- Never modify the org context block defaults without admin approval.
- Never use custom or AppSource visuals.
- Never publish to a workspace that was not confirmed by the user in Step 0.
- Never skip the design brief validation in Step 3.
- Canvas width is always 1650 px. Do not make it responsive or fluid.
- Always use Direct Lake storage mode — never switch to Import without explicit
  user instruction and admin sign-off.
- If the semantic model does not exist or is inaccessible, stop and surface the
  error clearly before proceeding.

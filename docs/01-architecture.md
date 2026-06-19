# Architecture

## Components

- **Dataverse** — system of record. 12 normalized tables, global Choices, relationships and column-level defaults. Business logic that must be enforced regardless of client lives here (required fields, defaults, status sets).
- **Power Apps (Canvas) — "Operations"** — the day-to-day app. Multi-screen, master-detail patterns, modal dialogs, reusable sidebar shell, premium visual system.
- **Power Automate** — server-side automation: uploading manual files to SharePoint and (roadmap) generating itinerary/budget PDFs and passport-expiry alerts.
- **SharePoint** — document library storing the physical files (vouchers, PDFs); Dataverse keeps the link and metadata.

## Data flow

```
            ┌─────────────────────────────┐
            │     Power Apps (Canvas)      │
            │        "Operations"          │
            └──────────────┬──────────────┘
                           │ Patch / Filter / LookUp
                           ▼
            ┌─────────────────────────────┐
            │          Dataverse           │
            │  Trips, Itineraries, Quotes  │
            │   Documents, masters, etc.   │
            └──────┬───────────────┬───────┘
                   │ trigger        │ read/write
                   ▼               │
            ┌───────────────┐      │
            │ Power Automate│──────┘
            └──────┬────────┘
                   │ create file / get link
                   ▼
            ┌───────────────┐
            │  SharePoint   │  vouchers, PDFs
            └───────────────┘
```

## Build order (incremental MVP)

1. Global Choices and the 12 Dataverse tables (masters → Trips → children → support).
2. Operations app core: Home, Trips list, Trip detail.
3. Passengers, itinerary versions, day-by-day builder, day editor.
4. Budget roll-up, supplier quotes, documents.
5. Hardening: delegation-safe filters, hotel logic, per-item quoting, billing overrides.
6. Automation (Power Automate) and the financial/portal phases.

## Visual system

A single reusable shell across every screen: fixed left sidebar (nav + branding), top bar (title via `Switch(App.ActiveScreen, …)`, user avatar), beige canvas, cream cards with rounded corners, terracotta accent, **Georgia** for headings and **Open Sans** for body. Classic controls are used where stylable color properties are required.

## Notable constraints handled

- **Source-code YAML parser quirks** — colon-space inside string values, block scalars, and record literals break the paste parser; handled by single-line values and applying record-literal logic via the formula bar.
- **Dataverse delegation** — `Filter`/`LookUp` on lookups aren't delegable; scoped by stable text keys for correctness on the data volumes involved.
- **Typed cards** — option-set and two-option fields require option-set members (not raw text/booleans) when compared or written.

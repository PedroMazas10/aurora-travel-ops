# Aurora Travel — Operations Management System (Microsoft Power Platform)

> End-to-end business application that replaces spreadsheets and email with a structured operations system for a boutique travel consultancy: trip planning, versioned itineraries, supplier quoting, budgeting and document management.

**Stack:** Dataverse · Power Apps (Canvas) · Power Fx · Power Automate · SharePoint
**Role:** Solution design & full build (data model, apps, automation)
**Status:** MVP in production use · iterating on financial & portal phases

> ⚠️ This is an **anonymized** case study. The client name, data and screenshots are fictional/sanitized. The architecture, data model and formula patterns reflect a real delivered solution.

---

## The problem

A travel consultancy ran its entire operation on disconnected spreadsheets and email:

- Trips, passengers and itineraries lived in separate files with no single source of truth.
- Itineraries were rebuilt from scratch for every trip, even for repeat destinations.
- Supplier quotes were tracked manually, with no link to the specific service being quoted.
- Budgets (cost vs. sale vs. margin) were recalculated by hand and drifted out of sync.
- Vouchers and PDFs were scattered across inboxes and drives.

## The solution

A normalized **Dataverse** model plus a premium-styled **Canvas App** (Operations) that walks the team through the full lifecycle:

**Trip → Passengers → Itinerary versions → Days → Activities → Supplier quotes → Budget → Documents**

Key capabilities:

- **Versioned itineraries** — preliminary / official / final versions per trip, with a reusable library of "official" itineraries you can clone to a new trip or client ("use as base").
- **Day-by-day builder** — each itinerary day holds typed activities (flights, hotels, transfers, excursions, safaris…) with cost, sale and margin.
- **Hotel logic** — when an activity is a hotel, conditional fields capture check-in/out, nights, nightly rate and room qty, and auto-compute the hotel total into the cost.
- **Per-item supplier quoting** — multiple quotes per activity; selecting a winner writes its cost and supplier back to the itinerary item and marks the rest as received.
- **Live budgeting** — cost/sale/margin roll up per day and per version in real time.
- **Document hub** — vouchers and generated PDFs, filtered by trip and type, with SharePoint storage via Power Automate.

## Architecture

```
Power Apps (Canvas) ──► Dataverse (12 tables, global Choices)
        │                      ▲
        │                      │
        └──► Power Automate ───┘ ──► SharePoint (files: vouchers, PDFs)
```

See [`docs/01-architecture.md`](docs/01-architecture.md) for the full picture and [`docs/02-data-model.md`](docs/02-data-model.md) for the entity-relationship diagram.

## What's in this repo

| Path | Contents |
|---|---|
| [`docs/01-architecture.md`](docs/01-architecture.md) | Components, data flow, design decisions |
| [`docs/02-data-model.md`](docs/02-data-model.md) | Dataverse tables, relationships, ERD (Mermaid) |
| [`docs/03-screens.md`](docs/03-screens.md) | Canvas app screens & navigation |
| [`docs/04-power-fx-highlights.md`](docs/04-power-fx-highlights.md) | Representative Power Fx patterns (sanitized) |
| [`docs/05-power-automate.md`](docs/05-power-automate.md) | Flow specs (file → SharePoint, PDF generation) |
| `assets/` | Screenshots (add your sanitized captures here) |

## Selected engineering highlights

- **Reusable itinerary cloning** — deep-copies a version's days and items into a new trip in a single Power Fx routine, remapping each item to its new day.
- **Robust delegation-aware filtering** — child records scoped by stable text keys instead of record equality (Dataverse can't compare lookup-to-record), avoiding silent data cross-contamination.
- **Conditional typed forms** — hotel fields show/compute only for hotel activities; a two-option field driven by a styled toggle via its option-set member.
- **Premium, consistent UI system** — fixed sidebar, terracotta/beige palette, Georgia + Open Sans, reused across every screen.

A few representative formulas live in [`docs/04-power-fx-highlights.md`](docs/04-power-fx-highlights.md).

## Tech stack

- **Microsoft Dataverse** — relational data model, global option sets, security
- **Power Apps Canvas** — Operations app (multi-screen, master-detail, modals)
- **Power Fx** — Patch / ForAll / Filter / LookUp / collections, form logic
- **Power Automate** — file upload to SharePoint, PDF generation
- **SharePoint** — document library storage

## Roadmap

- [x] MVP: Operations app (trips, itineraries, quotes, budget, documents)
- [ ] Finance app (margin dashboards, cost-by-supplier)
- [ ] Operational calendar (SharePoint)
- [ ] Power Automate: itinerary/budget PDF generation, passport-expiry alerts
- [ ] External client portal (Power Pages)

## Author

Built by **Pedro Mazas & Mateo Sartor** — Civil/Structural Engineer & BIM Specialist focused on workflow automation and business-application development.

## License

[MIT](LICENSE)

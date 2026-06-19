# Canvas App — Screens & Navigation

The Operations app is a multi-screen Canvas app. A reusable shell (sidebar + top bar) repeats on every screen; the title is driven by `Switch(App.ActiveScreen, …)`.

## Navigation map

```
Home ──► Trips list ──► Trip detail ──► Passengers
                              │
                              ├──► Itinerary versions ──► Itinerary builder (days)
                              │                                 │
                              │                                 └──► Day editor (activities)
                              │                                          └──► Item quotes
                              └──► Budget review

Sidebar ──► Itinerary library (official itineraries, reusable)
Sidebar ──► Documents (vouchers & PDFs)
```

## Screens

| Screen | Purpose | Key elements |
|---|---|---|
| **Home / Dashboard** | Entry point | KPI summary, quick nav |
| **Trips list** | Browse/search trips | Gallery, filters, "new trip" |
| **Trip detail** | One trip's overview | Master record, links to passengers / itineraries / budget |
| **Trip passengers** | Manage who travels | Inline add (passenger + role), rooming, per-passenger billing override |
| **Itinerary versions** | Versions of one trip | Gallery of versions, "new version", "duplicate" |
| **Itinerary library** | All official itineraries | Cross-trip gallery, filters, **"use as base"** clone |
| **Itinerary builder** | Days of a version | Day list with counts/totals, add/delete day |
| **Day editor** | Activities of a day | Master-detail: activity gallery + typed form, conditional hotel fields, "quotes" button |
| **Item quotes** | Quotes for one activity | Create/compare quotes, **pick winner** → writes cost & supplier back |
| **Budget review** | Version totals | Cost / Sale / Margin cards + per-day breakdown |
| **Documents** | Files hub | Card grid, filter by trip & type, upload (→ SharePoint), open/delete |

## UX patterns used

- **Master-detail** — gallery on the left, edit form on the right (days, quotes, documents).
- **Modals** — overlay + card + visibility flag (`gModalOpen`) for create/edit and delete confirmation.
- **Conditional fields** — form cards show/hide based on the selected `Item Type` (hotel block).
- **Write-back actions** — selecting a winning quote patches the parent itinerary item.

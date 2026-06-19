# Power Automate — Flows

## 1. Upload document file to SharePoint

**Trigger:** Dataverse — *When a row is added or modified* on `Documents`, with a trigger condition that fires only when a file is present and `File Link` is still empty (prevents loops).

**Actions:**
1. Download the file from the Dataverse File column.
2. Create the file in the trip's SharePoint document library.
3. Update the `Documents` row, writing the SharePoint URL into `File Link`.

**Result:** the app's "Open" button (`Launch(ThisItem.'File Link')`) works uniformly for uploaded vouchers and generated PDFs.

## 2. Generate itinerary / budget PDF (roadmap)

**Trigger:** button in the app (or version status change).
**Actions:** assemble the itinerary/budget into an HTML template → convert to PDF → store in SharePoint → create a `Documents` row (type `Itinerary PDF` / `Budget PDF`) with the link.

## 3. Passport-expiry alerts (roadmap)

**Trigger:** scheduled (daily).
**Actions:** query passengers whose passport expiration is within N days of an upcoming trip → notify operations.

## Conventions

- Prefix flow names by domain (`Doc-…`, `Trip-…`).
- Guard triggers with conditions to avoid loops.
- Configure run-after error handling on the final write step.

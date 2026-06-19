# Power Fx Highlights

Representative, sanitized patterns from the Operations app. Table/choice names are generic.

## 1. Clone an itinerary version into a new trip ("use as base")

Deep-copies a version's days and items into a destination trip, remapping each item to its newly created day. Collections are loaded first to avoid "can't operate on the same data source used in ForAll".

```powerfx
// Resolve destination trip + snapshot source data
ClearCollect(colSrcDays,  Filter('Itinerary Days',  'Itinerary Version'.'Version Name' = gSourceVersion.'Version Name'));
ClearCollect(colSrcItems, Filter('Itinerary Items', 'Itinerary Version'.'Version Name' = gSourceVersion.'Version Name'));

// New version
Set(gNewVer, Patch('Itinerary Versions', Defaults('Itinerary Versions'),
    {
        'Version Name': gDestTrip.'Trip Name' & " V" &
            (CountRows(Filter('Itinerary Versions', Trip.'Trip Name' = gDestTrip.'Trip Name')) + 1),
        Trip: gDestTrip,
        'Version Type': 'VersionType'.Preliminary,
        Currency: gSourceVersion.Currency
    }
));

// Copy days
ForAll(colSrcDays As d,
    Patch('Itinerary Days', Defaults('Itinerary Days'),
        { 'Day Title': d.'Day Title', 'Itinerary Version': gNewVer, Trip: gDestTrip,
          'Day Number': d.'Day Number', 'Sort Order': d.'Sort Order' }));
Refresh('Itinerary Days');

// Copy items, remapping each to the matching new day
ForAll(colSrcItems As it,
    Patch('Itinerary Items', Defaults('Itinerary Items'),
        { 'Item Title': it.'Item Title', 'Itinerary Version': gNewVer, Trip: gDestTrip,
          'Itinerary Day': LookUp('Itinerary Days',
              'Itinerary Version'.'Version Name' = gNewVer.'Version Name'
              && 'Day Title' = it.'Itinerary Day'.'Day Title'),
          'Item Type': it.'Item Type', 'Cost Amount': it.'Cost Amount', 'Sale Amount': it.'Sale Amount' }));
```

## 2. Delegation-safe child filtering

Dataverse can't compare a lookup to a record (`'Itinerary Item' = gSelectedItem` → *Incompatible types: Record, Record*), and day titles repeat across trips. Scope by **version + day number** (or a stable text key) instead:

```powerfx
Sort(
    Filter('Itinerary Items',
        'Itinerary Version'.'Version Name' = gVersion.'Version Name',
        'Itinerary Day'.'Day Number'      = gSelectedDay.'Day Number'),
    'Sort Order', SortOrder.Ascending
)
```

## 3. Conditional hotel logic (form)

Hotel-only fields appear and compute only when the activity type is a hotel:

```powerfx
// Card.Visible
DataCardValue_ItemType.Selected.Value = 'ItemType'.Hotel

// Nights (read-only)
If(IsBlank(dpCheckIn.SelectedDate) || IsBlank(dpCheckOut.SelectedDate), 0,
   DateDiff(dpCheckIn.SelectedDate, dpCheckOut.SelectedDate, TimeUnit.Days))

// Hotel Total (read-only) = nights * nightly rate * rooms
Value(txtNights.Text) * Value(txtRate.Text) * If(Value(txtRooms.Text) = 0, 1, Value(txtRooms.Text))

// Cost Amount card .Update — use the hotel total when it's a hotel
If(DataCardValue_ItemType.Selected.Value = 'ItemType'.Hotel,
   Value(txtHotelTotal.Text), Value(txtCost.Text))
```

## 4. Winning supplier quote → write back to the item

Selecting a winning quote marks the rest as received and pushes its cost & supplier onto the parent item:

```powerfx
Set(locWasSelected, ThisItem.'Quote Status' = 'QuoteStatus'.Selected);
If(locWasSelected,
    Patch('Supplier Quotes', ThisItem, { 'Quote Status': 'QuoteStatus'.Received }),
    UpdateIf('Supplier Quotes',
        'Itinerary Item'.'Item Title' = gSelectedItem.'Item Title' && 'Quote Name' <> ThisItem.'Quote Name',
        { 'Quote Status': 'QuoteStatus'.Received });
    Patch('Supplier Quotes', ThisItem, { 'Quote Status': 'QuoteStatus'.Selected });
    Patch('Itinerary Items', gSelectedItem,
        { 'Cost Amount': ThisItem.'Quoted Cost', Supplier: ThisItem.Supplier })
);
Refresh('Supplier Quotes'); Refresh('Itinerary Items')
```

## 5. Two-option field driven by a styled toggle

A Dataverse two-option (Yes/No) field rendered as a toggle needs an **option-set member**, not a boolean:

```powerfx
// Card .Update
If(Toggle1.Value, 'IncludeInBudget'.Yes, 'IncludeInBudget'.No)
```

## 6. Combo display workaround

When a Dataverse combo renders blank values, project a clean text column with the **display name** and bind `DisplayFields` to it:

```powerfx
// Items
AddColumns(Trips, TripLabel, 'Trip Name')
// DisplayFields / SearchFields
["TripLabel"]
```

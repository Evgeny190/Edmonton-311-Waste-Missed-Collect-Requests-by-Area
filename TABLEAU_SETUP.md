# Tableau Dashboard Setup

This file documents the Tableau configuration used for the Edmonton 311 missed-collection dashboard.

## 1. Connect the data

Connect:

- `data/processed/edmonton_missed_waste_collection_monthly.csv`
- `data/spatial/edmonton_neighbourhoods.geojson`

Create a relationship:

```text
CSV.neighbourhood_number = GeoJSON.neighbourhood_number
```

Logical cardinality: **Many → One**.

Set `neighbourhood_number` to **String** in both tables.

## 2. Map sheet

Suggested sheet name: **Missed Collections Map**

Marks:

- `COLLECT(Geometry)` — map geometry
- `Neighbourhood` — Detail
- `Requests Color` — Color
- `SUM(Requests)` — Tooltip
- `ATTR(Service Group)` — Tooltip
- `ATTR(Year)` — Tooltip
- `ATTR(Month Name)` — Tooltip

Do **not** place `Month Name` on Detail. Keeping it on Detail can make a neighbourhood polygon split into separate month-level marks and can interfere with dashboard filter actions.

Calculated field used during development:

```tableau
// Requests Color
IF SUM([Requests]) = 0 THEN
    NULL
ELSE
    SUM([Requests])
END
```

For a white-zero / red-higher map, assign the calculated field to Color and use a sequential red palette. Keep `SUM(Requests)` separately on Tooltip so zero requests can still display as `0` in the tooltip.

Suggested tooltip:

```text
<Neighbourhood>

Year: <ATTR(Year)>
Month: <ATTR(Month Name)>
Service: <ATTR(Service Group)>

Missed Collection Requests: <SUM(Requests)>
```

## 3. Monthly trend sheet

Suggested sheet name: **Monthly Trend**

- `Month Date` → Columns → **Month (Continuous)**
- `SUM(Requests)` → Rows
- Marks → **Line**
- `Service Group` → Color or Filter

Date format for `Month Date`: `mmmm yyyy`.

The trend should show monthly points such as January 2025, February 2025, etc., not annual totals.

The Month dashboard filter should **not** be applied to this sheet if the goal is to preserve the full within-year trend.

## 4. Top 10 neighbourhoods

Suggested sheet name: **Top 10 Neighbourhoods**

- `Neighbourhood` → Rows
- `SUM(Requests)` → Columns
- Marks → Bar
- `SUM(Requests)` → Label
- Sort descending by `SUM(Requests)`

Neighbourhood filter:

```text
Top 10 by SUM(Requests)
```

To make Top 10 respond correctly to dashboard selections, add these as **Context Filters**:

- Year
- Month / Month Date
- Service Group

If zero rows should be excluded from the ranking, filter using:

```text
SUM(Requests) >= 1
```

Do not use `MIN(Requests) >= 1`, because the zero-filled fact table can give a neighbourhood a minimum of zero even when it has requests elsewhere in the selected period.

## 5. Dashboard filters

Recommended user-facing controls:

- Year — Single Value (dropdown)
- Month — Single Value (dropdown)
- Service Group — Single Value (dropdown)

Apply filters as follows:

| Filter | Map | Top 10 | Monthly Trend |
|---|---:|---:|---:|
| Year | Yes | Yes | Yes |
| Service Group | Yes | Yes | Yes |
| Month | Yes | Yes | No |

## 6. Map → Trend interaction

Use a custom dashboard Filter Action rather than relying on the sheet's automatic `Use as Filter` button.

**Dashboard → Actions → Add Action → Filter**

Configuration:

```text
Source sheet: Missed Collections Map
Run action on: Select
Target sheet: Monthly Trend
Clearing selection: Show all values
Filter: Selected Fields
Source Field: Neighbourhood
Target Field: Neighbourhood
```

Only `Neighbourhood → Neighbourhood` should be passed. Using `All fields` can unintentionally pass month or other mark-level fields and collapse the trend to a single point.

## 7. Suggested dashboard layout

```text
┌─────────────────────────────────────────────────────────┐
│ Edmonton Missed Waste Collection — 311 Analysis       │
│ Year | Month | Service Group                           │
├──────────────────────────────┬──────────────────────────┤
│ Neighbourhood Map            │ Top 10 Neighbourhoods   │
├──────────────────────────────┴──────────────────────────┤
│ Monthly Missed Collection Requests                     │
└─────────────────────────────────────────────────────────┘
```

# ASAN Service Statistics: Interactive Power BI Dashboard

An interactive Power BI dashboard that analyzes **104.7 million** citizen service requests handled by ASAN service centers in Azerbaijan between **2013 and 2026** (206,234 records, 57 branches).

The dashboard combines an interactive map of Azerbaijan, time-based KPIs and slicers, wrapped in a dark, glassmorphism-style UI.

> **Screenshots:** add your images to `images/` and link them here.
>
> `![Dashboard overview](images/dashboard-overview.png)`

## Key features

- **Interactive map of Azerbaijan:** select regions or cities to cross-filter every other visual
- **Slicers:** year, service type (Əsas / Funksional) and center type (Mərkəz / Səyyar)
- **4 core KPIs:**
  - Total requests
  - Average daily requests
  - Number of branches
  - Annual growth rate (YoY %)
- **Custom theme:** dark, semi-transparent "glass" cards with rounded corners and a generated gradient background

## Key finding

About **62%** of all requests (65.3 M of 104.7 M) come from the **Baku-Absheron** region.

| Region (simplified grouping) | Total requests |
|---|---:|
| Baku-Absheron | 65,276,535 |
| Aran | 12,268,879 |
| Ganja-Gazakh | 10,495,402 |
| North-West | 4,225,605 |
| South | 3,744,098 |
| North-East | 3,710,430 |
| Location not assigned (mobile units, etc.) | 3,380,455 |
| Nakhchivan | 1,578,175 |
| Karabakh | 19,179 |
| **Total** | **104,698,758** |

## Dataset

`MainStatistics.csv`: daily request counts per ASAN branch.

| Column | Description |
|---|---|
| `appeal_count` | Number of requests (integer) |
| `date` | Date, `MM/DD/YYYY` format |
| `branch_name` | Branch name (57 unique values) |
| `service_type` | Service type: `Əsas` (main) or `Funksional` (functional) |
| `branch_mobile` | Center type: `Mərkəz` (fixed center) or `Səyyar` (mobile) |

- Period: 2013-01-15 to 2026-11-07 (4,706 distinct days)
- No missing values, no duplicate rows
- File encoding is **UTF-16**, tab-separated

## Data model

```
DimDate (1) ───────< (many) MainStatistics (many) >─────── (1) DimBranch
 Date                        date, branch_name,               branch_name, City,
 Year, Quarter,              appeal_count, ...                Region, Latitude,
 Month, ...                                                   Longitude
```

- **`MainStatistics`:** fact table
- **`DimDate`:** calendar table created with DAX and marked as a date table
- **`DimBranch`:** branch → city → region lookup with coordinates (`DimBranch.csv`)

### DimBranch notes

- Regions are a **simplified geographic grouping**, not the official economic regions of Azerbaijan.
- City coordinates are approximate city centers.
- All Baku ASAN centers (ASAN 1–7, ASAN kommunal 1–3) share one point on the map (Baku).
- 41 of 57 branches appear on the map. The remaining 16 have no fixed location (for example `Səyyar ASAN 1–10`, order and organization units, `ASAN Qatar`) and are grouped as "Location not assigned".

## DAX

```dax
DimDate =
ADDCOLUMNS(
    CALENDAR(DATE(2013, 1, 1), DATE(2026, 12, 31)),
    "Year", YEAR([Date]),
    "Quarter", "Q" & QUARTER([Date]),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMMM", "az-Latn-AZ"),
    "Year-Month", FORMAT([Date], "YYYY-MM")
)
```

```dax
Total Appeals = SUM(MainStatistics[appeal_count])

Branch Count = DISTINCTCOUNT(MainStatistics[branch_name])

Avg Daily Appeals =
DIVIDE([Total Appeals], DISTINCTCOUNT(MainStatistics[date]))

Appeals LY =
CALCULATE([Total Appeals], SAMEPERIODLASTYEAR(DimDate[Date]))

YoY % =
DIVIDE([Total Appeals] - [Appeals LY], [Appeals LY])
```

## How to rebuild

1. In Power BI Desktop, **Get data → Text/CSV** and open `MainStatistics.csv`. Set **File Origin** to `1200: Unicode` and **Delimiter** to `Tab`.
2. Open **Transform data**:
   - Set `date` to type **Date** using **Locale: English (United States)**
   - Apply **Format → Trim** on `branch_name`
3. Import `DimBranch.csv` (UTF-8, comma-separated). Set `Latitude` and `Longitude` to Decimal Number using the English (United States) locale, then mark them with the Latitude / Longitude data categories.
4. Create `DimDate` with the DAX above and mark it as a date table.
5. Create relationships: `DimDate[Date]` → `MainStatistics[date]` and `DimBranch[branch_name]` → `MainStatistics[branch_name]` (both one-to-many, single direction).
6. Add the measures above.
7. Apply `ASAN_Glass_Theme.json` (**View → Themes → Browse for themes**). Set the page size to 1920×1080 and use `ASAN_Background_1920x1080.png` as the canvas background.
8. Build the visuals: Map (Location = `City`, Legend = `Region`, Bubble size = `Total Appeals`), slicers and KPI cards.

> The Map visual must be enabled under **File → Options → Security → Map and filled map visuals**.

## Repository structure

```
.
├── README.md
├── data/
│   ├── MainStatistics.csv
│   └── DimBranch.csv
├── theme/
│   └── ASAN_Glass_Theme.json
├── assets/
│   └── ASAN_Background_1920x1080.png
├── images/
│   └── dashboard-overview.png
└── ASAN_Dashboard.pbix
```

## Limitations

- 2026 is a partial year (data ends on 2026-11-07), so the YoY % for 2026 is not comparable with full years.
- Regions are a simplified grouping and city coordinates are approximate.
- Glassmorphism is approximated with transparency, borders and shadows, because Power BI does not support real background blur.

## Tools

Power BI Desktop · Power Query · DAX · Data modeling · Data visualization

## Author

**Şamil Ağazadə (Shamil Aghazada)**, Junior Data Analyst, Baku, Azerbaijan

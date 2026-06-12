## Public health question

A simulated salmonella outbreak has produced a line list of ~200 patient addresses in Philadelphia. Where are cases clustering, and how can we map them without exposing any individual patient's location? Geocoding converts addresses to coordinates; privacy protection requires aggregating those points to a geography coarse enough that no single address can be inferred from the published map.

## What you will learn

- Inspect and clean an address CSV for geocoding
- Install the **MMQGIS** plugin and understand when to use it vs. the Census batch web form
- Batch geocode US addresses with the **US Census** service (free; generally acceptable to IRBs)
- Import geocoded coordinates with **Add Delimited Text Layer**
- Assess match rate and match type (Exact vs. interpolated vs. unmatched)
- Aggregate case points to census tracts with **Count points in polygon**
- Apply small-cell suppression with the **Field Calculator**
- Symbolize a suppressed choropleth using **Graduated** symbology

## What you will produce

1. A geocoded point layer of salmonella cases saved to a GeoPackage
2. A match-rate summary table (matched exact, interpolated, unmatched; percent matched)
3. A tract-level choropleth with cells fewer than 5 suppressed, exported as a Print Layout PDF
4. (Optional) A coarse-bandwidth kernel density surface
5. A ~100-word data-use rationale: why tracts, why suppress, what the analyst's working copy contains that the published map does not

## Data

All files are in `W:\Lab04b\data\`. The **W: drive** refers to the student network or USB storage location your instructor specified on Day 1.

| Layer | Description | Source |
|---|---|---|
| `outbreak_synthetic_addresses.csv` | ~200 synthetic patient addresses (no real PHI); fields: `CASE_ID`, `STREET`, `CITY`, `STATE`, `ZIP`, `ONSET_DATE`, `AGE_GRP`, `SEX` | Instructor-provided (generated from random Philadelphia rooftop points) |
| `phl_tracts.gpkg` | Philadelphia census tract boundaries, 2020 TIGER/Line, GeoPackage format | Instructor-provided (Census TIGER) |

> **Warning:** In an actual outbreak, patient addresses are **PHI**. Never upload real addresses to a public geocoding service, a cloud API, or a public AI tool. Keep identifiable data on secured, IRB-approved systems. The synthetic file used here contains no real patient data.

## Before you begin

**Prerequisites:** Labs 1–4a. Be comfortable adding data, running Processing tools, and using the Field Calculator.

**Software:** QGIS 3.34 LTR (free and open-source; no credits or license fee). Download from [qgis.org](https://qgis.org) if not installed.

**Plugin:** Go to **Plugins ▸ Manage and Install Plugins…**, search for **MMQGIS**, and click **Install Plugin**. A new **MMQGIS** menu appears after installation.

**Estimated time:** 2.5–3 hours.

**Project:** **Project ▸ New**, then **Project ▸ Save As…** → `Lab04b_Geocoding.qgz` in `W:\Lab04b\`.

---

## Part 1: Inspect and Clean the Address Table

Good geocoding starts with a consistently formatted address table. Messy abbreviations and missing ZIP codes reduce match rates.

1. Open `W:\Lab04b\data\outbreak_synthetic_addresses.csv` in a spreadsheet application (Excel, LibreOffice Calc, or similar).

2. Confirm the columns: `CASE_ID`, `STREET`, `CITY`, `STATE`, `ZIP`. The Census Geocoder requires each address component in a separate field.

3. Scroll through `STREET`. Note inconsistent abbreviations ("Street" vs. "St" vs. "St."), non-numeric ZIPs, and blank cells.

4. Make a working copy named `addresses_clean.csv` in `W:\Lab04b\`. Edit only this copy.

5. Standardize with Find & Replace: "Street" → "St", "Avenue" → "Ave", "Boulevard" → "Blvd", "Drive" → "Dr".

6. For rows missing a ZIP, enter `19000` as a placeholder (prevents the geocoder skipping the row).

7. Save as plain UTF-8 CSV with a header row.

*[Figure: addresses_clean.csv open in a spreadsheet showing CASE_ID, STREET, CITY, STATE, ZIP columns with consistent formatting]*

---

## Part 2: Geocode with MMQGIS or the Census Batch Web Form

You have two free, privacy-appropriate options. Both use the US Census Geocoder. Choose based on your situation.

> **Public health note:** The US Census Geocoder (`geocoding.geo.census.gov`) does not retain submitted addresses and is widely accepted by health department IRBs for geocoding patient-linked addresses. Commercial geocoders (Google Maps, Esri World Geocoding, HERE) transmit addresses to private servers; many IRB protocols prohibit this for PHI. Nominatim/OpenStreetMap **forbids heavy bulk use** and is not appropriate for PHI workflows. Always verify your chosen service against your IRB protocol and any applicable BAA before using real patient data.

### Option A: MMQGIS plugin (recommended for reproducibility)

MMQGIS calls the Census geocoder from within QGIS, creates the point layer, and writes a "not-found" file for unmatched rows.

1. Go to **MMQGIS ▸ Geocode ▸ Geocode CSV with Web Service**.

2. Set parameters:

   | Parameter | Value |
   |---|---|
   | **Input CSV File** | `W:\Lab04b\addresses_clean.csv` |
   | **Web Service** | US Census |
   | **Address** | `STREET` |
   | **City** | `CITY` |
   | **State** | `STATE` |
   | **Country/ZIP** | `ZIP` |
   | **Output File Name** | `W:\Lab04b\cases_geocoded.csv` |
   | **Not Found Output List** | `W:\Lab04b\not_found.csv` |

3. Click **OK**. MMQGIS writes matched rows (with `Latitude` and `Longitude`) to `cases_geocoded.csv` and unmatched rows to `not_found.csv`. It may add a point layer automatically; if not, proceed to Part 3.

*[Figure: MMQGIS Geocode CSV dialog with US Census selected and field mappings filled in]*

> **Note:** MMQGIS submits rows one at a time to the Census geocoder's single-address endpoint. For 200 rows this typically takes 2–5 minutes. Do not close QGIS while it runs.

### Option B: Census batch web form (fallback)

If MMQGIS is unavailable, use the Census Geocoder batch form at `https://geocoding.geo.census.gov/geocoder/locations/addressbatch`. The form requires a four-column CSV **with no header row**: `Unique ID, Street Address, City, State, ZIP`. Set **Benchmark** to **Public_AR_Current** and click **Get Results**. Save the result as `geocode_results.csv`. Output column 6 contains longitude and latitude as a comma-separated pair — split it into separate `LONGITUDE` and `LATITUDE` columns (**Data → Text to Columns** in Excel) and save as `geocode_results_split.csv`. Column 3 is match status (`Match`, `No_Match`, `Tie`); column 4 is match type (`Exact` or `Non_Exact`).

---

## Part 3: Import Geocoded Points into QGIS

1. Go to **Layer ▸ Add Layer ▸ Add Delimited Text Layer** (Ctrl+L).

2. Browse to `cases_geocoded.csv` (Option A) or `geocode_results_split.csv` (Option B). Set **Geometry definition** to Point coordinates, **X field** to `Longitude`/`LONGITUDE`, **Y field** to `Latitude`/`LATITUDE`, **Geometry CRS** to EPSG:4326. Click **Add**, then **Close**.

3. Rename the layer `cases_geocoded` in the **Layers** panel.

4. Save to a GeoPackage: right-click `cases_geocoded` → **Export ▸ Save Features As…** → Format: GeoPackage, file `W:\Lab04b\Lab04b.gpkg`, layer name `cases_geocoded`, CRS EPSG:2272. Click **OK**. Use this reprojected layer for all subsequent steps.

*[Figure: QGIS map canvas showing geocoded case points over Philadelphia; Add Delimited Text Layer dialog in the background]*

> **Note:** EPSG:2272 is in US survey feet. Keep this in mind when setting any distance or radius parameter in later steps.

---

## Part 4: Review Match Rate and Rematch Failures

Assess geocoder performance before drawing conclusions.

1. Open `not_found.csv` (Option A) or filter `geocode_results.csv` to `No_Match` and `Tie` rows (Option B).

2. Complete the match-rate table below (you will submit this):

   | Category | Count | Percent |
   |---|---|---|
   | Total submitted | | |
   | Matched — Exact | | |
   | Matched — Non_Exact (interpolated) | | |
   | Tie | | |
   | No_Match | | |
   | **Total matched** | | |

3. For unmatched rows: remove unit designators (e.g., "123 Main St Apt 2" → "123 Main St"); fix misspelled street names; correct wrong ZIPs. Re-submit and update the table.

> **Note:** Record addresses that remain unmatched. Do not fabricate coordinates. If unmatched rows cluster in one neighborhood, your map has a systematic spatial gap — note this in your methods.

> **Public health note:** "Non_Exact" (interpolated) means the geocoder estimated a position along a street segment from address-range midpoints rather than a rooftop. Positional error can reach 50–150 m in dense cities. Interpolated matches are adequate for tract-level aggregation; they are not adequate for sub-block analyses. Report match type distribution in any methods section.

---

## Part 5: Aggregate Cases to Census Tracts

Mapping individual case points risks re-identifying patients. Aggregating to tracts protects privacy while preserving the spatial signal needed for public health action.

1. Confirm `phl_tracts` from `W:\Lab04b\data\phl_tracts.gpkg` is in the **Layers** panel (**Layer ▸ Add Layer ▸ Add Vector Layer** if not). Reproject if needed: right-click → **Export ▸ Save Features As…** with CRS EPSG:2272.

2. Open the **Processing Toolbox** (Ctrl+Alt+T). Search for **Count points in polygon**. Set **Polygons** to `phl_tracts`, **Points** to `cases_geocoded` (EPSG:2272), **Count field name** to `CASE_COUNT`, and save the output to `W:\Lab04b\Lab04b.gpkg` as layer `tracts_cases`. Click **Run**.

3. Open the attribute table (F6) and confirm `CASE_COUNT` values look plausible.

> **Public health note:** Census tracts contain roughly 2,500–8,000 residents, which prevents any individual case address from being inferred from the map. Patients in an outbreak investigation have not consented to public mapping of their home location.

---

## Part 6: Small-Cell Suppression

A choropleth showing "3 cases in Tract X" can still expose individuals. Standard practice suppresses cells fewer than 5 before any public-facing product.

1. Open the attribute table of `tracts_cases` (F6).

2. Click the **Field Calculator** button (the abacus icon, or Ctrl+I).

3. Create a new Text field named `CASES_SUPP` (length 10). In the expression box, enter:

   ```
   CASE WHEN "CASE_COUNT" < 5 THEN '<5' ELSE to_string("CASE_COUNT") END
   ```

5. Click **OK**. Rows with 0–4 cases show `<5`; rows with 5 or more show the numeric count as a string.

6. Toggle editing off (the pencil icon) and save the layer.

> **Warning:** Do not publish a map showing `CASE_COUNT` directly. Always use `CASES_SUPP` or an equivalent suppressed field in any product shared outside the investigation team.

*[Figure: Attribute table of tracts_cases showing CASE_COUNT and CASES_SUPP columns; several rows display \<5 in the suppressed field]*

---

## Part 7: Symbolize the Choropleth and Export

1. Open `tracts_cases` **Properties ▸ Symbology**. Set render type to **Categorized**, **Value** to `CASES_SUPP`, and click **Classify**.

2. Assign the `<5` category a light gray fill — this signals suppression visually. For numeric categories, apply a sequential **YlOrRd** ColorBrewer ramp.

3. Add a basemap: in the **Browser** panel, expand **XYZ Tiles** and double-click **OpenStreetMap**. Drag it below `tracts_cases`.

4. Open a Print Layout: **Project ▸ New Print Layout…**, name it `Lab04b_Map`. Add a map frame (**Add Item ▸ Add Map**), legend (label the suppressed category "Suppressed (< 5 cases)"), scale bar, north arrow, and title/data-source label.

5. Export: **Layout ▸ Export as PDF…** → `Lab04b_Map.pdf`.

*[Figure: Print Layout showing Philadelphia tract choropleth with suppressed tracts in gray, legend, north arrow, and scale bar]*

---

## Part 8 (Optional): Kernel Density Surface

A kernel density surface shows where cases concentrate as a continuous raster. Use a large bandwidth so no individual address dominates the pattern.

1. In the **Processing Toolbox**, search for **Heatmap (Kernel Density Estimation)**. Set **Point layer** to `cases_geocoded` (EPSG:2272), **Radius** to `4921` (feet; ≈ 1,500 m), **Pixel size X/Y** to `328` (≈ 100 m), **Kernel shape** to Quartic, and **Output raster** to `W:\Lab04b\kd_cases.tif`. Click **Run**.

2. In the raster **Properties ▸ Symbology**, set **Singleband pseudocolor** with a sequential ramp (e.g., Magma). Set the minimum slightly above zero to suppress the empty background. Place the raster below the tract layer.

> **Public health note:** Report the bandwidth in your methods and state clearly that the surface does not represent individual locations. KDE supports hypothesis generation; it does not replace tract-level counts for public reporting.

---

## Deliverables

Submit the following by the end of Day 4.

1. **Exported map** (PDF): tract choropleth with suppressed cells, legend (with `<5` labeled), north arrow, scale bar, and a data source note.
2. **Match-rate table**: completed table from Part 4 including post-rematch totals.
3. **Data-use rationale** (~100 words): why tracts rather than blocks; why suppress cells fewer than 5; what the analyst's working copy contains that the published map does not; and whether this synthetic workflow would apply to real patient data and under what conditions.
4. **(Optional)** Exported kernel density map with the same cartographic elements.

Label any AI-assisted content clearly in your deliverables.

---

## Using AI in this lab

AI can scaffold two tasks here. Use it only with the synthetic data file — never with real patient addresses or case IDs.

**Task 1 — Address standardization regex.** Before geocoding, ask an AI assistant:

> "Write a Python regex that maps Street/St/St., Avenue/Ave, Boulevard/Blvd, and Drive/Dr to standard abbreviations in a pandas column using `.str.replace()`. Show me an example."

Use the result to help clean `addresses_clean.csv`. Review every output — regex has edge cases ("Strathmore Ave" must not lose the "Ave").

**Task 2 — Draft a geocoded-address handling SOP.** Ask:

> "Draft a one-page SOP for geocoding patient addresses in outbreak investigations. Include allowable services, data storage, and suppression thresholds. Assume PHI."

Critique the draft against your IRB guidance and CDC data-sharing policy. Note gaps or conflicts. Submit your annotated critique.

> **Warning — AI guardrails:** Use AI only with `outbreak_synthetic_addresses.csv`. Never paste real patient names, addresses, or case IDs into a public AI tool. AI scaffolds and drafts; it does not choose geocoding services, evaluate match quality, set suppression thresholds, or interpret the epidemiological meaning of a cluster — those are your decisions. Label all AI-assisted content in deliverables.

---

## ArcGIS equivalents

| QGIS (this lab) | ArcGIS Pro |
|---|---|
| **MMQGIS ▸ Geocode CSV with Web Service** (US Census) | **Geocode Addresses** tool (World Geocoding Service or Census locator) |
| **Layer ▸ Add Delimited Text Layer** (X/Y from CSV) | **XY Table To Point** |
| **Count points in polygon** (Processing Toolbox) | **Spatial Join** (Join one-to-one, Contains) |
| **Field Calculator** expression (`CASE WHEN … END`) | **Field Calculator** (Python 3: `"<5" if !Join_Count! < 5 else str(!Join_Count!)`) |
| **Heatmap (Kernel Density Estimation)** | **Kernel Density** (Spatial Analyst) |
| **Export ▸ Save Features As…** with target CRS | **Project** tool (Data Management) |
| **Project ▸ New Print Layout** | ArcGIS Pro **Layout** view |

The Census batch web form is browser-based and platform-independent. The matching ArcGIS Pro and ArcGIS Online versions of this lab are available for those platforms.

---

## Check your understanding

1. A colleague says the Census Geocoder is safe for real patient addresses because "it's run by the government." What questions should you ask before accepting this, and where would you find a definitive answer?

2. Your batch geocode returns 78% matched exact, 14% interpolated, and 8% unmatched. A supervisor asks whether the unmatched cases are randomly distributed or clustered in one neighborhood. How would you investigate this in QGIS, and why does it matter for the validity of your outbreak map?

3. You suppress a tract showing 4 cases to `<5`. A journalist files a public records request for the underlying case count data. Describe the tension between public records law and patient privacy, and one mechanism health departments use to navigate it.

4. What is the difference between an Exact and a Non_Exact geocode match? Give one analysis where the distinction matters and one where it probably does not.

5. MMQGIS offers both US Census and Nominatim/OpenStreetMap as geocoding services. Why is Nominatim generally not appropriate for geocoding patient addresses, even if the data are synthetic in a training context?

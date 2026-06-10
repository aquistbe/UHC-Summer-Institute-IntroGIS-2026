## Public health question

A simulated salmonella outbreak has produced a line list of ~200 patient addresses in Philadelphia. Where are cases clustering, and how can we map them without exposing any individual patient's location? Geocoding converts addresses to coordinates; privacy protection requires aggregating those points to a geography coarse enough that no single address can be inferred from the published map.

## What you will learn

- Inspect and clean an address table before geocoding
- Contrast the ArcGIS Online World Geocoder (credits, cloud) with the Census Geocoder (free, privacy-safer)
- Add geocoded coordinates to ArcGIS Online as a CSV with X/Y fields
- Assess match results and match type (rooftop vs. interpolated)
- Aggregate case points to tracts with **Aggregate Points**
- Apply small-cell suppression with **Calculate** (Arcade)
- Style a suppressed choropleth and share a web map

## What you will produce

1. A geocoded point layer of synthetic salmonella cases shared to your org account
2. A match-rate summary table (exact, interpolated, unmatched; percent matched)
3. A tract-level choropleth with counts < 5 suppressed, saved and shared as a web map
4. A ~100-word data-use rationale (why tracts, why suppress, what the analyst's copy contains that the public map does not)
5. (Optional) A coarse-cell kernel density map

## Data

| Layer | Description | Source |
|---|---|---|
| `outbreak_synthetic_addresses.csv` | ~200 synthetic patient addresses (no real PHI); fields: `CASE_ID`, `STREET`, `CITY`, `STATE`, `ZIP`, `ONSET_DATE`, `AGE_GRP`, `SEX` | Instructor-shared item in your ArcGIS organizational account |
| `geocode_results_coords.csv` | Census Geocoder output reformatted with `LONGITUDE` and `LATITUDE` columns; you produce this in Part 2 | Your own work product |
| Philadelphia census tracts | 2020 TIGER/Line tract boundaries for Philadelphia County | Instructor-shared feature layer in your ArcGIS organizational account |

> **Warning:** Patient addresses are **PHI**. Do NOT upload real patient addresses to ArcGIS Online or the ArcGIS World Geocoding Service — transmitting PHI to a commercial cloud service may violate your IRB protocol and your organization's BAA with Esri. The synthetic file here contains no real patient data. For real data, use the Census Geocoder path in Part 2 and confirm with your IRB first.

## Before you begin

**Prerequisites:** Labs 1–4a. Be comfortable adding layers, running Analysis tools, and opening the attribute table in Map Viewer. **Estimated time:** 2.5–3 hours.

**Account:** Sign in to **arcgis.com** using the **ArcGIS organizational account** provided by the course. A free public account cannot run the Analysis tools used in this lab.

> **Note — Credits and privileges:** **Aggregate Points** and **Calculate Density** consume credits. The ArcGIS World Geocoding Service costs roughly 40 credits per 1,000 addresses. This lab uses the free Census Geocoder for geocoding and only brings coordinates into ArcGIS Online, avoiding that charge. Credits are spent only for the aggregation and optional density steps.

---

## Part 1: Inspect and Clean the Address Table

Good geocoding starts with consistently formatted addresses. Do this work in a spreadsheet application before uploading anything.

1. Download `outbreak_synthetic_addresses.csv` from the instructor-shared item in your ArcGIS organizational account. Open it in Excel or a text editor.

2. Confirm the columns: `CASE_ID`, `STREET`, `CITY`, `STATE`, `ZIP`. The Census Geocoder requires each address component in a separate field.

3. Make a working copy named `addresses_clean.csv`. Edit this copy only — keep the original intact.

4. Use Find & Replace to standardize street type abbreviations: "Street" → "St", "Avenue" → "Ave", "Boulevard" → "Blvd", "Drive" → "Dr". Check each substitution — "Strathmore Avenue" must become "Strathmore Ave", not damage the root word.

5. For rows missing a ZIP, enter `19000` as a placeholder (Philadelphia ZIPs begin 190–191). A missing ZIP causes the geocoder to skip the row.

6. Save as a plain UTF-8 CSV with the header row intact.

*[Figure: addresses_clean.csv open in Excel showing the five required columns with consistent street-type abbreviations and no blank ZIPs]*

> **Public health note:** Address cleaning reduces cases that appear "missing" — a form of spatial selection bias. If unmatched addresses cluster in one neighborhood, your map undercounts cases there. Document your decisions.

---

## Part 2: Geocode with the U.S. Census Geocoder

The ArcGIS Online World Geocoding Service can geocode your CSV directly inside Map Viewer, but it transmits addresses to Esri's servers — which many IRB protocols prohibit for patient-linked data. The **Census Geocoder** (`geocoding.geo.census.gov`) is free, US-only, and widely accepted as a privacy-safer alternative; submitted addresses are not retained. Use the Census path for real patient data, and confirm with your own IRB before using any external geocoder.

> **Public health note:** Any geocoding service receives addresses in cleartext. Confirm your IRB protocol and agency data governance policies before geocoding real patient data with any external service.

1. The Census Geocoder requires a four-column CSV with **no header row**: `Unique ID, Street Address, City, State, ZIP`. Remove the header row from `addresses_clean.csv` before upload.

2. Go to `https://geocoding.geo.census.gov/geocoder/locations/addressbatch`. Keep the **Address Batch** tab selected.

3. Click **Choose File**, upload your header-free CSV, select **Benchmark: Public_AR_Current**, and click **Get Results** (30–90 seconds for 200 rows).

4. Save the result as `geocode_results.csv`.

*[Figure: Census Geocoder batch interface with file uploaded and Public_AR_Current benchmark selected]*

5. Open `geocode_results.csv`. Key columns: Column 1 = `CASE_ID`; Column 3 = match status (`Match`, `No_Match`, `Tie`); Column 4 = match type (`Exact` or `Non_Exact`); Column 6 = longitude,latitude pair in WGS84.

6. Split Column 6 into `LONGITUDE` and `LATITUDE`. In Excel: **Data → Text to Columns**, delimiter = comma. Join these columns back to `addresses_clean.csv` on `CASE_ID` and save as `geocode_results_coords.csv`.

7. Tally your results (submit with deliverables):

    | Category | Count | Percent |
    |---|---|---|
    | Total submitted | | |
    | Matched — Exact | | |
    | Matched — Non_Exact (interpolated) | | |
    | Tie | | |
    | No_Match | | |
    | **Total matched** | | |

> **Public health note:** `Non_Exact` (interpolated) means the geocoder estimated a position along a street segment from address ranges, not a rooftop. Positional error can reach 50–150 m in dense cities. Report match-type distribution in any methods section.

---

## Part 3: Attempt Failed Addresses

The Census Geocoder has no interactive rematch interface. Handle unmatched rows manually.

1. Filter `geocode_results.csv` to `No_Match` and `Tie` rows. Copy them (with `CASE_ID`) to `addresses_rematch.csv`.

2. For each row: remove unit numbers from the street field; correct misspelled street names; fix wrong ZIPs.

3. Re-upload to the Census Geocoder. Save as `geocode_rematch.csv`, merge newly matched rows into `geocode_results_coords.csv`, and update the match-rate table.

> **Note:** Record addresses that remain unmatched. Do not fabricate coordinates. Document persistent failures and exclude them from spatial analysis with a methods note.

---

## Part 4: Add Geocoded Points to ArcGIS Online

ArcGIS Online can read a CSV with latitude and longitude columns and place points on the map without consuming geocoding credits.

1. Sign in to **arcgis.com** with your organizational account. Open **Map Viewer** from the app launcher (grid icon). Create a new map and **Save** it as `Lab04b_Geocoding_[YourInitials]` with the tag `IntroGIS`.

2. In the **Layers** pane (left toolbar), click **Add** → **Add layer from file**. Upload `geocode_results_coords.csv`.

3. ArcGIS Online detects `LONGITUDE` and `LATITUDE`. Confirm **Longitude field** = `LONGITUDE`, **Latitude field** = `LATITUDE`, **Coordinate system** = `WGS84`. Click **Next** → **Add to map**.

*[Figure: ArcGIS Online "Add layer from file" dialog showing LONGITUDE and LATITUDE field assignments for geocode_results_coords.csv]*

4. Open the layer's **Table** and confirm `CASE_ID`, `STREET`, and `ONSET_DATE` are present. In **Styles** (right toolbar), set the drawing style to **Location (Single Symbol)** with a small high-contrast circle.

> **Note:** You do not reproject in ArcGIS Online. Analysis tools compute geodesic distances, so aggregation to tracts is correct regardless of display projection — unlike ArcGIS Pro, where forgetting to reproject before a buffer introduces measurable error.

---

## Part 5: Aggregate Cases to Census Tracts

Mapping individual case points risks re-identifying patients. Aggregating to census tracts protects privacy while preserving the spatial pattern of the outbreak.

> **Warning:** **Aggregate Points** consumes credits. Review the estimated cost in the tool pane before clicking **Run**. For ~200 points and ~380 Philadelphia tracts, the cost is typically under 5 credits.

1. Confirm both layers are loaded. If the tracts layer is missing, click **Add** → **Browse layers** → **My Organization** and search for the instructor-shared tracts item.

2. Click **Analysis** → search for **Aggregate Points** and open the tool.

3. Set parameters:

    | Parameter | Value |
    |---|---|
    | Point layer | `geocode_results_coords` |
    | Polygon layer | Philadelphia census tracts |
    | Result layer name | `tracts_cases_[YourInitials]` |

4. Click **Run**. The output contains a field named `Point Count` with the case count per tract.

*[Figure: ArcGIS Online Analysis pane showing Aggregate Points tool with case points and tracts configured]*

> **Public health note:** Census tracts contain roughly 2,500–8,000 residents, making individual addresses unidentifiable in the published map. Patients have not consented to public mapping of their home location.

---

## Part 6: Small-Cell Suppression

A choropleth showing "3 cases in Tract X" can expose individuals. Suppress cells with fewer than 5 cases before sharing outside the investigation team.

1. Open the **Fields** pane (right toolbar) for `tracts_cases`. Click **Add field**: name `CASES_SUPP`, type `String`. Click **Save**.

2. Click **Calculate** on the `CASES_SUPP` field. Enter this Arcade expression (Arcade is ArcGIS Online's built-in scripting language — no external script required):

    ```
    if ($feature["Point Count"] < 5) {
      return "<5"
    } else {
      return Text($feature["Point Count"])
    }
    ```

    Click **Run**. Tracts with 0–4 cases show `<5`; others show the numeric count as text.

    > **Note:** `Point Count` (with a space) is the default field name from **Aggregate Points**. Check the **Fields** tab to confirm.

3. In **Styles**, choose `CASES_SUPP` and **Types (Unique Symbols)**. Assign light gray to `<5`; apply a sequential yellow-to-red ramp to numeric values. Set the `<5` legend label to "Suppressed (< 5 cases)".

*[Figure: Philadelphia tract choropleth with suppressed tracts in light gray and case counts 5+ in a yellow-to-red ramp]*

> **Warning:** Do not share any layer or map that exposes raw `Point Count` to external audiences. Use `CASES_SUPP` only. Set sharing to **Organization**, not **Everyone**, unless your supervisor explicitly approves public release.

---

## Part 7 (Optional): Kernel Density Surface

A kernel density surface shows concentration as a smooth continuous surface. Use a large search radius so no individual address dominates the pattern.

> **Warning:** **Calculate Density** consumes credits. Review the estimated cost before clicking **Run**.

1. Click **Analysis** → **Calculate Density**. Set **Input layer** to the case points, **Cell size** to `200` meters, **Search radius** to `1500` meters, and **Output layer name** to `kd_cases_[YourInitials]`. Click **Run**.

2. Move the density layer beneath the tract choropleth in the **Layers** pane.

> **Public health note:** A kernel density map can imply precision that does not exist. Include a methods note with the bandwidth and state that the surface must not be used to infer individual case locations.

---

## Deliverables

Submit the following by the end of Day 4:

1. **Shared web map URL:** save and share `Lab04b_Geocoding_[YourInitials]` to your organization showing the suppressed choropleth. Copy the item URL from **Share → Copy link**.
2. **Match-rate table:** completed table from Part 2, Step 7, including post-rematch totals.
3. **Exported map image:** screenshot showing the suppressed choropleth with a visible legend and scale reference. For a more polished product, build a short **ArcGIS StoryMap** or **Instant App** — ArcGIS Online has no full print layout editor like ArcGIS Pro.
4. **Data-use rationale** (~100 words): why tracts rather than blocks; why suppress cells fewer than 5; what the analyst's working copy contains that the published map does not; whether this synthetic workflow would apply to real patient data and under what conditions.
5. **(Optional)** Screenshot or web map showing the kernel density layer.

Label any AI-assisted content in your deliverables.

---

## Using AI in this lab

Use AI only with the synthetic data file — never with real patient addresses.

**Task 1 — Address standardization regex.** Ask Claude:

> "Write a Python regex using `pandas .str.replace()` that maps Street/St/St., Avenue/Ave, Boulevard/Blvd, and Drive/Dr to standard abbreviations. Include a test case that could fail — for example, a street name that contains 'Avenue' as part of its name."

Use the result to clean `addresses_clean.csv`. Review outputs before applying at scale — regex has edge cases.

**Task 2 — Draft a geocoded-address handling SOP.** Ask Claude:

> "Draft a one-page SOP for a local health department for geocoding patient addresses in outbreak investigations. Include: allowable geocoding services, data transmission requirements, storage and retention rules, and suppression rules before publication. Assume addresses are PHI."

Read the draft critically. Identify two claims that are vague, conflict with CDC's *Summary of Notifiable Diseases* data-sharing guidance, or would not satisfy standard IRB language. Submit your annotated critique.

> **Warning — AI guardrails:** Use AI only with `outbreak_synthetic_addresses.csv`. Never paste real patient names, addresses, or case IDs into a public AI tool. AI scaffolds and drafts; it does not choose geocoding services, evaluate match quality, or set suppression thresholds — those are your decisions. Label all AI-assisted content in deliverables.

---

## ArcGIS Pro equivalent

The matching Pro lab (Lab 4b, ArcGIS Pro) is available for desktop users. Key tool equivalents:

| ArcGIS Online | ArcGIS Pro |
|---|---|
| Add layer from file (CSV with X/Y fields) | **XY Table To Point** |
| No reprojection needed; analysis is geodesic | **Project** tool to local CRS before distance analysis |
| **Aggregate Points** | **Spatial Join** (target = tracts, one-to-one, count) |
| **Calculate** with Arcade | **Field Calculator** (Python 3) |
| **Calculate Density** | **Kernel Density** (Spatial Analyst) |
| Share web map; StoryMaps/Instant Apps for layout | **Layout** view; export to PDF |

The Census Geocoder workflow is browser-based and identical regardless of GIS platform.

---

## Check your understanding

1. A colleague says the ArcGIS World Geocoding Service is safe for real patient addresses because your organization has a license with Esri. What additional agreements or reviews would you confirm first, and where would you look for a definitive answer?

2. Your batch geocode returns 78% Exact, 14% Non_Exact (interpolated), and 8% No_Match. A supervisor asks whether unmatched cases are randomly distributed or clustered in one neighborhood. How would you investigate this, and why does it matter for the validity of your outbreak map?

3. You suppress a tract with 4 cases to `<5`. A journalist files a public records request for the underlying count data. Describe the tension between public records law and patient privacy, and name one mechanism health departments use to navigate it.

4. What is the practical difference between a rooftop (Exact) geocode and an interpolated (Non_Exact) geocode? Give one analysis where the distinction matters and one where it probably does not.

5. ArcGIS Online Analysis tools compute geodesic distances regardless of display projection. Explain why this is an advantage over a desktop workflow where a user forgot to reproject before running a buffer, and describe the error that desktop map would contain.

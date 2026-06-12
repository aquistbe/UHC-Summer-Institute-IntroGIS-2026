## Public health question

Diabetes burden and limited primary-care access often cluster in the same neighborhoods, compounding health risk. Which Philadelphia census tracts have **both** high adult diabetes prevalence **and** low primary-care provider access — and what is the racial/ethnic and economic composition of those tracts? Identifying this intersection is a core step in health-equity analysis and resource-allocation planning.

---

## What you will learn

- Add a GeoPackage spatial layer and CSV tables (no geometry) via **Add Delimited Text Layer**
- Inspect field types to confirm the join key is stored as text
- Join multiple CSV tables to a polygon layer with **Properties ▸ Joins** on a shared `GEOID` key
- Find percentile cutoffs with **Basic Statistics for Fields**
- Select features using a two-condition Boolean expression (**Select by Expression**)
- Export a selection to a new GeoPackage layer (**Save Features As**)
- Build a composite need index with the **Field Calculator** using QGIS expressions
- Symbolize qualifying versus non-qualifying tracts and produce a **Print Layout** PDF

---

## What you will produce

1. A QGIS project file (`Lab03a.qgz`) with the tract layer, joined tables, and symbology.
2. A PDF map exported from the **Print Layout** showing qualifying tracts highlighted.
3. A summary table comparing demographic composition of qualifying versus non-qualifying tracts.
4. A short written interpretation (~100 words) describing the spatial pattern and at least one data caveat.

QGIS is free and open-source — no licenses or course credits required.

---

## Data

Instructor-provided files in the course data folder (exact path shown on the classroom projector).

| Layer / table | Description | Source |
|---|---|---|
| `phl_tracts.gpkg` | TIGER/Line 2022 census tracts, Philadelphia County | U.S. Census Bureau |
| `PLACES_diabetes_tract.csv` | Adult diabetes crude prevalence (%), tract-level modeled estimates | CDC PLACES 2023 |
| `primarycare_access_tract.csv` | Primary-care providers per 10,000 residents, tract-level | HRSA Area Health Resource Files |
| `acs_tract.csv` | Race/ethnicity and median household income, 2018–2022 ACS 5-year | U.S. Census Bureau via NHGIS |

All three CSV files join to `phl_tracts.gpkg` on `GEOID` (11-digit tract FIPS code).

> **Note:** `GEOID` values in CSVs are sometimes stored as integers with leading zeros stripped. If the join produces many `NULL` rows, check that both join fields are type **String** (text), not Integer. Part 1, Step 4 shows how.

---

## Before you begin

**Prerequisites:** Lab 1 (QGIS interface) and Lab 2a (projections). You should be comfortable adding data and opening the **Attribute Table**.

**QGIS version:** QGIS 3.34 LTR (free at qgis.org). No plugins required for this lab.

**Time estimate:** 2.5–3 hours.

**Setup:** Open QGIS, create a new project (**Project ▸ New**), and save it as `Lab03a.qgz` in your course data folder.

---

## Part 1: Add data and inspect fields

1. In the **Browser** panel, navigate to your course data folder and double-click `phl_tracts.gpkg`. Philadelphia's 384 tract polygons appear on the canvas.

2. Go to **Layer ▸ Add Layer ▸ Add Delimited Text Layer**. Browse to `PLACES_diabetes_tract.csv`. Set **File format** to CSV, confirm **First record has field names** is checked, and under **Geometry definition** choose **No geometry (attribute only table)**. Click **Add**. Repeat for `primarycare_access_tract.csv` and `acs_tract.csv`. You now have one spatial layer and three CSV tables in the **Layers** panel.

   *[Figure: Layers panel listing phl_tracts.gpkg and the three CSV tables below it]*

3. Right-click `PLACES_diabetes_tract` in the **Layers** panel and choose **Open Attribute Table** (F6). Go to **Layer ▸ Properties ▸ Fields** and find the `GEOID` row. The **Type** column must read **String**. If it reads Integer or Double, notify the instructor — a corrected file will be provided.

4. Note the exact field names you will use in later expressions: `diab_crude_pct` (diabetes prevalence), `pc_per_10k` (primary-care access per 10k), and from `acs_tract.csv`: `pct_black_nh`, `pct_hispanic`, `pct_white_nh`, `med_hh_income`.

> **Public health note:** CDC PLACES estimates use multilevel regression and poststratification, not direct survey counts. They carry uncertainty for small tracts. Treat this analysis as a screening tool, not a definitive prevalence measure.

---

## Part 2: Join the CSV tables to the tract layer

You will perform three sequential joins. QGIS joins via **Properties ▸ Joins** are temporary — the source files are unchanged, and the join is stored in the project file.

1. Double-click `phl_tracts` in the **Layers** panel to open **Layer Properties**, then click **Joins** in the left column.

2. Click the green **+** button. In the **Add Vector Join** dialog:

   | Parameter | Value |
   |---|---|
   | Join layer | `PLACES_diabetes_tract` |
   | Join field | `GEOID` |
   | Target field | `GEOID` |
   | Cache join layer in virtual memory | Checked |

   Click **OK**, then **Apply**.

3. Repeat Step 2 for `primarycare_access_tract` and `acs_tract`, each joining on `GEOID` → `GEOID`. Click **OK** to close Layer Properties.

4. Open the **Attribute Table** of `phl_tracts` and scroll right to confirm that fields from all three CSV tables appear. QGIS prepends the source table name (e.g., `PLACES_diabetes_tract_diab_crude_pct`). Many `NULL` values in joined columns signals a type mismatch on the key field — re-check Step 3 in Part 1.

   > **Warning:** This join is temporary and requires the CSV files to remain at the same path. You will make the selection results permanent by exporting in Part 4.

> **Public health note:** All tables join on the **11-digit census tract GEOID** — not a ZIP code or ZCTA. ZCTAs approximate ZIP codes but do not align with tract boundaries. Joining ZCTA-coded data directly to tract polygons produces incorrect results. If your data source provides ZCTAs, you need a ZCTA-to-tract crosswalk.

---

## Part 3: Find percentile cutoffs

Before writing the selection query, find the 75th-percentile diabetes value and the 25th-percentile primary-care access value.

1. Go to **Vector ▸ Analysis Tools ▸ Basic Statistics for Fields**. Set **Input layer** to `phl_tracts` and **Field** to `PLACES_diabetes_tract_diab_crude_pct`. Leave output as a temporary layer and click **Run**. In the Results Viewer, record the **Q3** (75th percentile) value.

2. Run the tool again with field set to `primarycare_access_tract_pc_per_10k`. Record the **Q1** (25th percentile) value.

   | Statistic | Value (fill in) |
   |---|---|
   | Diabetes prevalence, 75th percentile (Q3) | _______ % |
   | Primary-care access, 25th percentile (Q1) | _______ per 10,000 |

   *[Figure: Basic Statistics results panel showing Q1, Median, Q3 for diab_crude_pct]*

   > **Note:** The **Statistics** panel (**View ▸ Panels ▸ Statistics**) shows quartiles live as you switch fields — a quick alternative to running the tool.

---

## Part 4: Select by Expression and export

1. Click `phl_tracts` in the **Layers** panel to make it active. Open its **Attribute Table** (F6) and click the **Select features using an expression** button (yellow ε icon).

2. In the expression box, enter the following, replacing the placeholders with your recorded values:

   ```
   "PLACES_diabetes_tract_diab_crude_pct" > 14.2
   AND
   "primarycare_access_tract_pc_per_10k" < 3.1
   ```

   In QGIS expressions, field names are in **double quotes**. Click **Select Features**. Expect roughly 15–30 tracts selected. Close the dialog.

   *[Figure: Map canvas with selected tracts highlighted in yellow against gray unselected tracts]*

   > **Public health note:** This AND query identifies tracts above the 75th percentile for diabetes AND below the 25th percentile for primary-care access — a joint threshold more conservative than flagging either condition alone. This is still an **ecological analysis**: tract-level results describe places, not individuals. Attributing tract-level statistics to individuals is the **ecological fallacy**. The choice of administrative boundary also shapes which tracts qualify (**Modifiable Areal Unit Problem, MAUP**). Use this map to generate hypotheses, not causal conclusions.

3. Right-click `phl_tracts` ▸ **Export ▸ Save Selected Features As**. Set format to **GeoPackage**, name the file `high_need_tracts.gpkg`, layer name `high_need_tracts`, and confirm that only selected features will be saved. Click **OK**. The new permanent layer appears in the **Layers** panel.

4. Clear the selection: **Edit ▸ Deselect Features from All Layers**.

---

## Part 5: Calculate a composite need index

The instructor has pre-calculated `diab_pctl` (diabetes percentile rank, 0–100) and `pcaccess_inv_pctl` (inverted primary-care access rank, 0–100) in `high_need_tracts`. Higher values mean greater need.

1. Open the **Attribute Table** of `high_need_tracts`. Click the **Field Calculator** button (abacus icon).

2. Set **Create a new field**, name it `need_index`, type **Decimal number (double)**. In the expression box:

   ```
   ("diab_pctl" + "pcaccess_inv_pctl") / 2
   ```

   Click **OK**. Enable editing if prompted. Scroll the `need_index` column to verify values fall between 0 and 100. Save edits and toggle editing off.

---

## Part 6: Compare qualifying versus non-qualifying tracts

1. Open the **Attribute Table** of `phl_tracts` and toggle editing on. Open the **Field Calculator**; create a new **Text (string)** field named `qual_group` (length 20) with the expression `'Not qualifying'`. Apply to all records (uncheck "Only update selected features"). Click **OK**.

2. Re-run the **Select by Expression** from Part 4 with the same cutoff values. Then re-open the **Field Calculator**, choose **Update existing field**, set field to `qual_group`, check **Only update selected features**, and enter `'Qualifying'`. Click **OK**. Clear the selection, save edits, and toggle editing off.

3. Run **Processing ▸ Toolbox ▸ Statistics by categories** for each demographic variable (`acs_tract_pct_black_nh`, `acs_tract_pct_hispanic`, `acs_tract_pct_white_nh`, `acs_tract_med_hh_income`). Set **Field with categories** to `qual_group`. Record the group means in a table like the one below.

   | Variable | Qualifying (mean) | Not qualifying (mean) |
   |---|---|---|
   | % Black non-Hispanic | | |
   | % Hispanic | | |
   | % White non-Hispanic | | |
   | Median household income ($) | | |

---

## Part 7: Symbolize and export the map

1. Double-click `phl_tracts` ▸ **Properties ▸ Symbology**. Choose **Categorized**, set **Value** to `qual_group`, click **Classify**. Assign a neutral gray to `Not qualifying` and a high-contrast color (orange or red) to `Qualifying`. Select both classes, right-click ▸ **Change Symbol**, set **Stroke style** to **No Pen**. Click **Apply**.

2. Drag an **OpenStreetMap** basemap from **Browser ▸ XYZ Tiles** to the bottom of the **Layers** panel for geographic context.

3. Go to **Project ▸ New Print Layout**. Add a map frame (**Add Item ▸ Add Map**), then add a **Title**, **Legend**, **Scale Bar**, and **North Arrow**. Export via **Layout ▸ Export as PDF** at 150 dpi, saving as `Lab03a_map.pdf`.

   *[Figure: Print Layout with Philadelphia tracts, qualifying tracts in orange, legend, scale bar, north arrow]*

---

## Deliverables

1. **PDF map** with qualifying tracts highlighted (`Lab03a_map.pdf`).
2. **Summary table** comparing demographic composition (Part 6).
3. **Written interpretation** (~100 words): where do qualifying tracts cluster, which groups are over-represented, and what caution would you add for a community partner audience? Include at least one reference to MAUP or ecological fallacy.
4. Label any AI-assisted steps on your submitted work.

---

## Using AI in this lab

### Use 1 — Field Calculator expression

Ask Claude:

> "I am using QGIS 3.34 Field Calculator. I have fields `diab_pctl` and `pcaccess_inv_pctl`, each 0–100. Write a QGIS expression to average them into a new field `need_index`."

Expected response: `("diab_pctl" + "pcaccess_inv_pctl") / 2`

Verify the result: all values should be between 0 and 100.

### Use 2 — Translating an ArcGIS expression to QGIS

Ask Claude:

> "Translate this ArcGIS Pro Field Calculator Python expression to QGIS syntax: `(!diab_pctl! + !pcaccess_inv_pctl!) / 2`."

Expected response: `("diab_pctl" + "pcaccess_inv_pctl") / 2` — the only change is replacing `!field!` notation with `"field"` notation.

> **AI guardrail:** AI is useful for scaffolding expressions, explaining errors, and critiquing layout. It does **not** choose data, set thresholds, or interpret results — those are your analytic decisions. Never paste patient identifiers or individually identifiable health data into a public AI tool. Label AI-assisted output on your submission.

---

## ArcGIS equivalents

The ArcGIS Pro and ArcGIS Online versions of this lab use the same public health question and deliver the same products via Esri tools.

| QGIS | ArcGIS Pro |
|---|---|
| **Add Delimited Text Layer** (No geometry) | **Add Data** — CSV table in Contents |
| **Properties ▸ Joins ▸ green +** | **Joins and Relates ▸ Add Join** |
| **Select by Expression** (ε icon) | **Select by Attributes** |
| **Basic Statistics for Fields** / **Statistics by categories** | **Summary Statistics** with optional Case Field |
| **Field Calculator** — field names in `"double quotes"` | **Calculate Field** Python — field names as `!field!` |
| **Export ▸ Save Selected Features As** (GeoPackage) | **Data ▸ Export Features** (file geodatabase) |
| **Symbology ▸ Categorized** | **Symbology ▸ Unique Values** |
| **Project ▸ New Print Layout** | **Insert ▸ New Layout** |

---

## Check your understanding

1. What is the difference between a **one-to-one** and a **one-to-many** join? Give a public health example of each.

2. After the join, many rows show `NULL` in `diab_crude_pct`. Name two possible causes and explain how to diagnose each.

3. In plain language, what does the `AND` operator do in your selection expression? How would the selected set change if you used `OR` instead?

4. A colleague wants to announce a new clinic location based solely on your qualifying-tracts map. What two analytical cautions — one about the data, one about the geography — would you raise?

5. Why is the ZCTA-to-tract join issue a practical problem? Describe a scenario where confusing ZCTAs with census tracts would produce a misleading result.

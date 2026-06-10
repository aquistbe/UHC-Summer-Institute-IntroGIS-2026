## Public health question

Diabetes burden and limited access to primary care often cluster together in the same neighborhoods, compounding health risk. This lab asks: which Philadelphia census tracts have **both** high adult diabetes prevalence **and** low primary-care provider access? And what is the racial/ethnic and economic composition of those tracts? Identifying the intersection of disease burden and access barriers is a core step in health-equity analysis and resource-allocation planning.

---

## What you will learn

- Add a hosted feature layer (census tracts) and CSV tables to ArcGIS Online Map Viewer
- Use the **Join Features** analysis tool to join tabular data to a spatial layer on a shared key field (`GEOID`)
- Understand one-to-one join cardinality and the ZCTA ≠ ZIP Code trap
- Find percentile cutoffs using the **Statistics** panel and layer charts
- Build a two-condition Boolean query with **Filter**
- Save a filtered subset as a new hosted feature layer
- Write an **Arcade** expression in **Fields > Calculate** to build a composite need index
- Compare group statistics between qualifying and non-qualifying tracts
- Style and share a web map that communicates the results

---

## What you will produce

1. A **saved and shared web map** of Philadelphia census tracts with qualifying tracts (high diabetes prevalence AND low primary-care access) highlighted in a high-contrast color.
2. A **screenshot or exported image** of the map view for submission, captured with the browser's Print or the map's Share function.
3. A **short written interpretation** (approximately 100 words) describing the spatial pattern, the demographic and income composition of qualifying tracts, and at least one caveat about the data or analysis.

> **Note:** ArcGIS Online does not have a full print layout editor like ArcGIS Pro. To produce a shareable map product, use **Save and Share** on your web map, then capture the view with the browser's print-to-PDF function, or publish it as an **ArcGIS Instant App** or **StoryMap** for a polished deliverable.

---

## Data

| Layer / table | Description | Source / access method |
|---|---|---|
| `phl_tracts` | TIGER/Line 2022 census tracts, Philadelphia County (hosted feature layer) | Instructor-shared item in your ArcGIS organizational account |
| CDC PLACES — Census Tracts | Adult diabetes crude prevalence (%), tract-level modeled estimates | **ArcGIS Living Atlas of the World** (search "CDC PLACES") |
| `primarycare_access_tract.csv` | Primary-care providers per 10,000 residents, tract-level | Instructor-provided file (download from course portal) |
| `acs_tract.csv` | Race/ethnicity proportions and median household income, 2018–2022 ACS 5-year | Instructor-provided file (download from course portal) |

All tables join to `phl_tracts` on the field `GEOID` (the 11-digit tract FIPS code, e.g., `42101000100`).

> **Note:** CDC PLACES tract-level data is available directly from the **ArcGIS Living Atlas**. Using the Living Atlas layer avoids file-type issues and keeps the data current. You will still join the primary-care and ACS tables from instructor-provided CSV files.

---

## Before you begin

**Account:** Sign in to **arcgis.com** using your **ArcGIS organizational account** provided by the course. Do not use a free public ArcGIS account — free accounts cannot run Analysis tools.

> **Warning — credits and privileges:** Running **Analysis** tools (including **Join Features**) consumes **credits** from your organizational account and requires **publisher** or **analysis** privileges. Each Join Features run on a large layer costs credits. Be deliberate: set all parameters before clicking **Run**, and do not run the same tool repeatedly. If you see a message saying you lack privileges, contact the instructor before proceeding.

**Prerequisites:** You should have completed Lab 1 (Map Viewer orientation) and Lab 2a (coordinate systems). You should be comfortable opening a web map, adding a layer, and using the **Contents** and **Settings** toolbars.

**Time estimate:** 2.5–3 hours.

**Setup:** Open a browser and go to [arcgis.com](https://www.arcgis.com). Sign in. From the app launcher (grid icon, top right), choose **Map Viewer**. Click **New map**. Immediately click **Save** (the floppy-disk icon in the left toolbar), give your map the title `Lab03a_YourLastName`, add the tag `IntroGIS`, and click **Save**.

---

## Part 1: Add the tracts layer and the CDC PLACES data

### Add the Philadelphia tracts layer

1. In the **Contents** toolbar (left side, dark background), click **Layers**, then click **Add** (the plus icon).

2. Choose **Browse layers**. In the search bar at the top of the browse panel, switch the search scope to **My Organization**. Search for `phl_tracts`. Click the result shared by your instructor, then click **Add**. Philadelphia's 384 census tracts appear as polygons on the map.

   *[Figure: Map Viewer showing Philadelphia tract polygons with the Layers panel open on the left]*

### Add CDC PLACES from Living Atlas

3. Click **Add** again and choose **Browse layers**. Switch the search scope to **Living Atlas**. Search for `CDC PLACES census tracts`. Select the layer titled **PLACES: Local Data for Better Health, Census Tract Data** (published by Esri/CDC). Click **Add**.

   > **Note:** The Living Atlas layer covers the entire United States. ArcGIS Online will display it at the national extent initially. Zoom to Philadelphia — the tracts will draw with prevalence data already attached. You do not need to join this table; the attributes are already in the layer. You will use it to extract the `DIABETES_CrudePrev` field value when setting your filter cutoff in Part 3.

4. In the **Layers** panel, click the three-dot menu next to the PLACES layer and choose **Show table**. Scroll to find the `DIABETES_CrudePrev` column (adult diabetes crude prevalence, %). Confirm the column is populated for Philadelphia tracts.

5. Click **Save** to preserve the current map state.

---

## Part 2: Join the CSV tables to the tracts layer

You will use the **Join Features** analysis tool to attach the primary-care access and ACS demographic data to the `phl_tracts` layer. Each join is **one-to-one**: one CSV row matches one tract polygon on the shared `GEOID` field.

> **Warning — credits:** Each **Join Features** run consumes credits. You will run this tool twice (once for the primary-care table, once for the ACS table). Confirm your parameters carefully before clicking **Run** each time.

### Prepare and upload the CSV files

1. Download `primarycare_access_tract.csv` and `acs_tract.csv` from the course portal to your computer.

2. In Map Viewer, click **Add → Add layer from file**. Drag `primarycare_access_tract.csv` into the upload area or browse to it. ArcGIS Online will detect it as a table (no geometry). Click **Next**, confirm the field types, then click **Add**. The table appears in the **Layers** panel with a spreadsheet icon. Repeat for `acs_tract.csv`.

   > **Note:** Before joining, open each table's **Table** view (three-dot menu → **Show table**) and inspect the `GEOID` column. Confirm the values are 11-digit strings like `42101000100`, not truncated integers. If leading zeros are missing (you see values like `42101100` instead of `42101000100`), the join will silently fail to match most records. Alert your instructor — a corrected file will be provided.

   > **Note — ZCTA ≠ ZIP Code ≠ Census Tract:** All three tables in this lab join on the **11-digit census tract GEOID** (state FIPS + county FIPS + tract code). This is not the same as a ZIP code or a ZIP Code Tabulation Area (ZCTA). ZCTAs are Census Bureau constructs that approximate ZIP codes but do not align with tract boundaries and cannot be directly joined to tract data using a ZIP field. If you ever obtain a data table keyed to ZIP codes and need to join it to tracts, you need a ZCTA-to-tract crosswalk file — a direct join will produce misleading or empty results.

### Run Join Features (primary-care access)

3. In Map Viewer, click **Analysis** (the wrench icon in the left toolbar). Search for `Join Features` and open the tool.

4. Set the following parameters:

   | Parameter | Value |
   |---|---|
   | Target layer | `phl_tracts` |
   | Join layer | `primarycare_access_tract` (the uploaded table) |
   | Join type | **One to one** |
   | Join operation | **Join one to one** |
   | Define join fields | Target field: `GEOID` · Join field: `GEOID` |
   | Keep all target features | Yes (checked) |
   | Output layer name | `phl_tracts_pcjoin` |

   > **Note:** "Keep all target features" (the equivalent of a LEFT JOIN in SQL) ensures every tract polygon is retained even if no matching row exists in the CSV. Unmatched rows will show `<Null>` in the joined fields. If you see many nulls after the join, the `GEOID` fields did not match — check field type and leading zeros.

5. Click **Estimate credits** to see the projected cost, then click **Run**. The output layer `phl_tracts_pcjoin` is added to your map and saved to **My Content**.

   *[Figure: Join Features tool pane showing the parameter settings before running]*

### Run Join Features (ACS demographics)

6. Open **Join Features** again. This time, set:

   | Parameter | Value |
   |---|---|
   | Target layer | `phl_tracts_pcjoin` (the output from Step 5) |
   | Join layer | `acs_tract` (the uploaded table) |
   | Join type | **One to one** |
   | Define join fields | Target field: `GEOID` · Join field: `GEOID` |
   | Keep all target features | Yes |
   | Output layer name | `phl_tracts_joined` |

7. Click **Run**. The final joined layer `phl_tracts_joined` now contains original tract geometry, PLACES diabetes data (from Part 1), primary-care access (`pc_per_10k`), and ACS race/ethnicity and income fields (`pct_white_nh`, `pct_black_nh`, `pct_hispanic`, `pct_asian_nh`, `med_hh_income`).

8. Open the table of `phl_tracts_joined` (three-dot menu → **Show table**) and scroll across all columns to confirm all fields are present. Close the table. Click **Save**.

---

## Part 3: Find percentile cutoffs

Before writing the filter query, you need to know the 75th-percentile diabetes prevalence value and the 25th-percentile primary-care access value for Philadelphia tracts.

1. In the **Layers** panel, click `phl_tracts_joined` to make it active.

2. Open the **Statistics** panel: click the three-dot menu next to the layer → **View table** → click the column header for `DIABETES_CrudePrev` → select **Statistics**. A summary panel appears showing minimum, maximum, mean, and a histogram. Read the approximate 75th percentile from the histogram or from the quantile display.

   > **Note:** ArcGIS Online's built-in statistics panel shows summary statistics but does not directly display a named "75th percentile" value. Use this approach: open the **Charts** option (three-dot menu → **Create chart → Histogram**), set the field to `DIABETES_CrudePrev`, and set the number of bins to 4 (quartile breaks). The break between the third and fourth bin approximates the 75th percentile. Alternatively, your instructor may provide the cutoff values directly.

3. Repeat for `pc_per_10k` to find the 25th-percentile primary-care access value (the break between the first and second bin in a 4-bin histogram).

4. Record the values here — you will need them in Part 4:

   | Statistic | Value (fill in) |
   |---|---|
   | Diabetes prevalence, 75th percentile | _______ % |
   | Primary-care access, 25th percentile | _______ per 10,000 |

   *[Figure: Histogram chart for DIABETES_CrudePrev showing approximate quartile breaks]*

---

## Part 4: Filter to qualifying tracts

**Filter** in ArcGIS Online is the web equivalent of Select by Attributes in ArcGIS Pro. It builds a Boolean expression that shows only the rows meeting your conditions.

1. With `phl_tracts_joined` active, click **Filter** in the **Settings** toolbar (right side, light background).

2. Click **Add expression**. Build the first clause:

   | Part | Value |
   |---|---|
   | Field | `DIABETES_CrudePrev` |
   | Operator | `is greater than` |
   | Value | *(enter your 75th-percentile value from Part 3)* |

3. Click **Add expression** again. The operator joining the two expressions defaults to **And** — leave it as **And**. Build the second clause:

   | Part | Value |
   |---|---|
   | Field | `pc_per_10k` |
   | Operator | `is less than` |
   | Value | *(enter your 25th-percentile value from Part 3)* |

4. Click **Save**. The map now displays only the tracts meeting both conditions — expect roughly 15–30 tracts. All other tracts are hidden (not deleted; the filter is non-destructive).

   *[Figure: Map showing a subset of Philadelphia tracts passing the two-condition filter, highlighted against a muted basemap]*

   > **Public health note:** This Boolean AND query identifies tracts simultaneously above the 75th percentile for diabetes AND below the 25th percentile for primary-care access. Even so, this is an **ecological analysis**: the result describes census tracts, not individuals. A tract above the 75th percentile for diabetes does not mean every resident has diabetes, and a resident of a low-access tract may still reach a provider in a neighboring tract. Interpreting tract-level results as descriptions of individuals is the **ecological fallacy**. Additionally, because tract boundaries are administrative artifacts, the choice of boundary affects which tracts qualify — a phenomenon called the **Modifiable Areal Unit Problem (MAUP)**. Use these maps to generate hypotheses and prioritize outreach, not to draw firm causal conclusions. CDC PLACES estimates are modeled (using multilevel regression and post-stratification), not directly measured, and carry uncertainty — particularly for small tracts.

5. Click the three-dot menu on `phl_tracts_joined` → **Save as layer**. Name it `high_need_tracts`. This creates a permanent hosted feature layer containing only the qualifying tracts. The filter on `phl_tracts_joined` can now be cleared (click **Filter → Remove filter**) to restore all tracts for comparison.

---

## Part 5: Calculate a composite need index

Add a numeric composite index that averages diabetes need and primary-care access need into a single score for each qualifying tract.

> **Note:** The instructor has pre-calculated percentile-rank columns `diab_pctl` and `pcaccess_inv_pctl` in the joined layer. Both range from 0 to 100, and higher values indicate greater need. If these columns are not present, ask your instructor for updated data.

1. In the **Layers** panel, click `high_need_tracts` to make it active.

2. In the **Settings** toolbar (right side), click **Fields**. The field list for the layer opens.

3. Click **Add field** (the plus icon). Set:

   | Setting | Value |
   |---|---|
   | Field name | `need_index` |
   | Display name | `Need Index` |
   | Type | `Double` |

   Click **Create**. The new field appears at the bottom of the field list with all values `<Null>`.

4. Click the three-dot menu next to `need_index` and choose **Calculate**. An Arcade expression editor opens.

5. In the expression box, type the following **Arcade** expression exactly:

   ```
   ($feature["diab_pctl"] + $feature["pcaccess_inv_pctl"]) / 2
   ```

   Click **OK**. ArcGIS Online runs the calculation for all records in the layer. Scroll through the table to confirm `need_index` values are populated between 0 and 100 with no nulls.

   > **Note:** ArcGIS Online field calculations use **Arcade**, not Python. In Arcade, field values are accessed with `$feature["field_name"]` (bracket notation) or `$feature.field_name` (dot notation). Standard arithmetic operators apply. This is different from ArcGIS Pro's Field Calculator Python mode, where field names are wrapped in exclamation marks (e.g., `!diab_pctl!`).

6. Click **Save** to save the field changes.

---

## Part 6: Compare group statistics

Use the **Statistics** and **Charts** panels to compare the demographic composition and median income of qualifying tracts against the Philadelphia average.

1. With `phl_tracts_joined` active (filter removed, all 384 tracts visible), open the **Table** (three-dot menu → **Show table**). Note the citywide mean of `med_hh_income` by clicking the column header → **Statistics**.

2. Now open the table for `high_need_tracts`. Click the `med_hh_income` column header → **Statistics**. Compare the mean for qualifying tracts to the citywide mean recorded in Step 1.

3. Repeat Steps 1–2 for `pct_black_nh`, `pct_hispanic`, and `pct_white_nh` to compare racial/ethnic composition.

4. Record your findings in the table below for your written interpretation:

   | Measure | All Philadelphia tracts (mean) | Qualifying tracts (mean) |
   |---|---|---|
   | Median household income | | |
   | % Non-Hispanic Black | | |
   | % Hispanic | | |
   | % Non-Hispanic White | | |

   *[Figure: Statistics panel for a demographic field showing the comparison between the full layer and the filtered qualifying layer]*

---

## Part 7: Style and share the web map

1. In the **Layers** panel, ensure both `phl_tracts_joined` (all tracts) and `high_need_tracts` (qualifying tracts) are visible and `high_need_tracts` is listed above `phl_tracts_joined`.

2. Click `phl_tracts_joined` to make it active. In the **Settings** toolbar, click **Styles**. Under **Choose attributes**, select `qual_group` (or, if that field is not present, select **Location (single symbol)**). Set the fill to a neutral light gray (e.g., `#D3D3D3`) and the outline to `No color`. Click **Done**.

3. Click `high_need_tracts` to make it active. Click **Styles**. Choose **Location (single symbol)** — since every tract in this layer qualifies, a single high-contrast color is appropriate. Set the fill to a strong orange or red (e.g., `#E05B0E`). Set the outline to a slightly darker shade of the same color at 1 pt. Click **Done**.

4. In the left toolbar, click **Labels** and add a label on `high_need_tracts` using the tract `GEOID` or `NAME` field, font size 8, white text. Click **Done**.

5. Click **Save**.

6. To share the map: in the left toolbar, click **Share** (the person-with-arrow icon). Set sharing to **Your organization** (or **Everyone** if instructed). Copy the item URL — this is your shareable deliverable.

   *[Figure: Final web map showing all Philadelphia tracts in light gray with qualifying high-need tracts highlighted in orange]*

---

## Deliverables

Submit the following on the course portal:

1. **Web map URL:** the shareable link to your saved map (`Lab03a_YourLastName`) set to organizational or public sharing.
2. **Exported map image or PDF:** use your browser's Print → Save as PDF while the map is displayed, or use the **Print** widget if your instructor has added one to the map. Ensure qualifying tracts are clearly visible.
3. **Filled comparison table** from Part 6 (median income, racial/ethnic proportions for all tracts vs. qualifying tracts).
4. **Written interpretation** (approximately 100 words): Where are the qualifying tracts located in Philadelphia? What racial/ethnic groups and income levels are over-represented? What caution would you add if presenting this map to a community partner? Include at least one reference to MAUP or ecological fallacy.
5. **Label your deliverables** if you used AI assistance in any step (see below).

---

## Using AI in this lab

Two bounded uses of AI are appropriate here.

### Use 1 — Generating the Arcade Calculate expression

If you are unsure how to write the Arcade expression for `need_index`, provide Claude with your field names and the desired math:

> "I am using ArcGIS Online Fields > Calculate with Arcade. I have two fields in a hosted feature layer: `diab_pctl` (diabetes percentile rank, 0–100) and `pcaccess_inv_pctl` (inverted primary-care access percentile rank, 0–100). Write an Arcade expression that averages them into a new field called `need_index`."

Claude should return something equivalent to:

```
($feature["diab_pctl"] + $feature["pcaccess_inv_pctl"]) / 2
```

Paste this into the **Calculate** expression editor. Verify that all `need_index` values fall between 0 and 100 with no nulls before moving on.

### Use 2 — Translating a Pro Python Field Calculator expression to Arcade

If you have a Python expression from the ArcGIS Pro version of this lab and need the Arcade equivalent, ask Claude:

> "Translate this ArcGIS Pro Field Calculator Python expression to an ArcGIS Online Arcade expression: `(!diab_pctl! + !pcaccess_inv_pctl!) / 2`. The layer has the same two field names."

Claude should return:

```
($feature["diab_pctl"] + $feature["pcaccess_inv_pctl"]) / 2
```

The key difference: ArcGIS Pro Python mode uses `!field_name!` (exclamation marks); Arcade uses `$feature["field_name"]` (bracket notation referencing the current feature object).

> **AI guardrail:** AI tools are useful for scaffolding expressions, explaining error messages, and critiquing your map design. They do **not** make analytic decisions for you — the choice of the 75th and 25th percentile cutoffs, the selection of variables, and the interpretation of results are yours. Never paste patient identifiers, addresses, or any individually identifiable health data into a public AI tool. AI-generated outputs should be verified against the data before you trust them. If you use AI assistance, note it explicitly on your submitted deliverables (e.g., "Arcade expression generated with Claude AI, verified by student").

---

## ArcGIS Pro equivalent

The ArcGIS Online workflow in this lab parallels the Pro desktop workflow, but uses different tools and terminology.

| ArcGIS Online step | ArcGIS Pro equivalent |
|---|---|
| **Add layer from file** (CSV) | **Add Data** → CSV; appears as standalone table in Contents |
| **Join Features** (Analysis tool) | **Add Join** (right-click layer → Joins and Relates) |
| **Filter** (Settings toolbar) | **Select by Attributes** (Map tab → Selection group) |
| **Save as layer** from filter | **Export Features** → new feature class |
| **Fields > Calculate** (Arcade) | **Field Calculator** (Python 3 or Arcade mode) |
| **Statistics** panel / Charts | **Summary Statistics** geoprocessing tool |
| **Share** web map + browser Print | **Export Layout** → PDF (Insert → New Layout) |

One key difference: ArcGIS Online joins via **Join Features** create a **new output layer** (a permanent hosted feature layer in My Content). ArcGIS Pro **Add Join** creates a **temporary in-memory join** that must be made permanent by exporting. The credit cost and output permanence should inform which platform you choose for a given workflow.

The matching ArcGIS Pro lab is **Lab 3a (ArcGIS Pro): Joining and Querying Public Health Surveillance Data**, which covers the same analysis using desktop geoprocessing tools.

---

## Check your understanding

1. In **Join Features**, you set the join type to **One to one**. What would happen if the primary-care CSV contained two rows for the same tract GEOID — for example, if a tract had data from two different provider databases? How would ArcGIS Online handle the duplicate, and what should you do before running the join?

2. After running **Join Features**, you open the table of the output layer and find that 80 of 384 tracts have `<Null>` in the `pc_per_10k` column. List two possible causes and explain how you would diagnose each.

3. The **Filter** tool in ArcGIS Online hides non-matching features but does not delete them. How is this different from **Save as layer** from the filter result? Describe a situation in a public health workflow where you would want one behavior but not the other.

4. You have applied the diabetes AND primary-care filter and identified 22 qualifying tracts. A colleague proposes presenting this map at a community meeting as evidence that those tracts "have a diabetes problem." Using the concepts of MAUP and the ecological fallacy, explain two cautions you would raise before that presentation.

5. The CDC PLACES layer is available directly from the ArcGIS Living Atlas, while the primary-care and ACS tables require a CSV upload. What are two practical advantages of using a Living Atlas layer over a locally uploaded file in an organizational GIS workflow?

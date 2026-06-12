## Public health question

Philadelphia hosts dozens of facilities that report toxic chemical releases to the EPA's Toxic Release Inventory (TRI). Residents living near these facilities may face elevated exposure risk — but who lives nearby, and are those populations disproportionately composed of people of color? This lab uses buffer zones and overlay to estimate the population within 0–500 m, 500 m–1 km, and 1–2 km of TRI facilities and compares the racial/ethnic composition of that population to the rest of the city.

---

## What you will learn

- Confirm a layer's CRS is projected in meters before running distance operations
- Create multi-distance buffer rings using **Multi-ring buffer (constant distance)**
- Overlay buffer rings with census block groups using **Intersection**
- Calculate fragment area and apportion block-group population using the **Field Calculator**
- Select block groups within 1 km using **Extract/Select by location**
- Isolate the "outside 1 km" set using **Difference**
- Compare demographic composition inside versus outside 1 km using **Join attributes by location (summary)** or **Basic statistics for fields**
- Symbolize buffer rings and export a print layout in QGIS

---

## What you will produce

1. A QGIS project file (`Lab03b_TRI_Buffer.qgz`) containing all layers and symbology.
2. A Print Layout PDF (`Lab03b_TRI_map.pdf`): three concentric buffer rings (0–500 m, 500 m–1 km, 1–2 km) over block groups symbolized by percent non-White population, with TRI facility points, legend, scale bar, north arrow, and data credits.
3. A completed comparison table (inside vs. outside 1 km) with percent calculations, pasted into a Word document or submitted as a screenshot.
4. A 100-word written interpretation of the comparison table noting at least two analytic limitations.
5. An AI-corrected Methods paragraph (see "Using AI in this lab").

---

## Data

| Layer | Description | Source |
|---|---|---|
| `phl_TRI_facilities.gpkg` | EPA Toxic Release Inventory facilities, Philadelphia subset | Instructor-provided; original data from EPA TRI Explorer (epa.gov/tri) |
| `phl_blockgroups_acs.gpkg` | ACS 2019–2023 5-year block groups with race/ethnicity table B03002 | Instructor-provided; original data from Census Bureau / NHGIS |

Both files are GeoPackage format (`.gpkg`) and are pre-projected to **NAD83 UTM Zone 18N (EPSG:26918)**, map units in meters. Files are in the course shared folder your instructor directed you to. You will confirm the CRS in Part 1 before running any distance operations.

---

## Before you begin

**Prerequisites:** Completion of Labs 1, 2a, and 3a. You should know how to add data, open the Processing Toolbox, run tools, open attribute tables, and use the Field Calculator.

**Software:** QGIS 3.34 LTR. QGIS is free and open-source — no licenses, credits, or account required. Download from qgis.org if not already installed.

**No plugins required for this lab.** All tools used are native to QGIS 3.34.

**Estimated time:** 2.5–3 hours.

Open QGIS and create a new project: **Project ▸ New**. Immediately save it: **Project ▸ Save As…**, name it `Lab03b_TRI_Buffer.qgz`, and save to your working folder.

---

## Part 1: Confirm the CRS and load data

> **Note:** Distance-based operations — buffers, proximity analysis — require a projected CRS with linear units in meters. Running a buffer in a geographic CRS (degrees) will produce incorrect distances. Always confirm before proceeding.

1. Add the data. Use **Layer ▸ Add Layer ▸ Add Vector Layer**, or press **Ctrl+L** to open the Data Source Manager. Navigate to your course data folder and add `phl_TRI_facilities.gpkg` and `phl_blockgroups_acs.gpkg`. Both layers will appear in the **Layers** panel.

2. Check the CRS of `phl_TRI_facilities`. Right-click the layer in the **Layers** panel and choose **Properties**. Click the **Information** tab. Under **Coordinate Reference System**, confirm it reads **NAD83 / UTM zone 18N** and that the units are **meters**. Click **OK**.

3. Repeat step 2 for `phl_blockgroups_acs`. Both layers must show meters.

> **Warning:** If either layer shows a geographic CRS — such as WGS 84 (EPSG:4326) with units in degrees — do not proceed with buffering. Reproject the layer first: right-click the layer → **Export ▸ Save Features As…**, set the CRS to **EPSG:26918**, save the output, and use that reprojected file going forward. Buffers run in a geographic CRS appear to complete successfully but the distances are wrong — QGIS measures in degrees, not meters.

4. Check the project CRS: look at the bottom-right of the QGIS window. It should show **EPSG:26918**. If it shows something else, click that button and set the project CRS to EPSG:26918 (**Project ▸ Properties ▸ CRS**, search `26918`, select **NAD83 / UTM zone 18N**, click **OK**).

5. Zoom to the data extent: right-click `phl_blockgroups_acs` in the **Layers** panel → **Zoom to Layer**.

*[Figure: Map canvas showing TRI facility points over Philadelphia block group polygons; CRS indicator in the status bar reads EPSG:26918]*

6. Open the attribute table for `phl_blockgroups_acs`: press **F6** or right-click the layer → **Open Attribute Table**. Identify these fields:

   | Field | Contents |
   |---|---|
   | `GEOID` | Block group FIPS code (unique identifier) |
   | `TOTPOP` | Total population (ACS B03002_001E) |
   | `NH_WHITE` | Non-Hispanic White alone (B03002_003E) |
   | `NH_BLACK` | Non-Hispanic Black or African American alone (B03002_004E) |
   | `HISPANIC` | Hispanic or Latino of any race (B03002_012E) |
   | `NH_OTHER` | All other non-Hispanic groups combined |
   | `AREASQM` | Block group area in square meters (pre-calculated) |

   Close the attribute table.

---

## Part 2: Create multi-distance buffer rings

> **Public health note:** Proximity to a TRI facility is a crude exposure proxy. People closer to a facility may face higher potential exposure, but wind direction, facility operating hours, chemical properties, and individual behavior all affect actual exposure. This analysis screens for potential inequity — it does not measure dose or establish causation.

1. Open the Processing Toolbox: **Processing ▸ Toolbox** (or **Ctrl+Alt+T**).

2. In the search box, type `multi-ring` and double-click **Multi-ring buffer (constant distance)** (under **Vector geometry**).

3. Set parameters:

   | Parameter | Value |
   |---|---|
   | Input layer | `phl_TRI_facilities` |
   | Number of rings | `3` |
   | Distance between rings | `500` (meters) |
   | Output | Save to your working folder as `TRI_rings_3.gpkg` |

4. Click **Run**. Three concentric rings appear: 0–500 m, 500–1000 m, and 1000–2000 m around the combined extent of all TRI facilities. Each ring is a separate feature with a `ringId` field (values 1, 2, 3).

   > **Note:** **Multi-ring buffer (constant distance)** creates nested rings where ring 1 spans 0–500 m, ring 2 spans 0–1000 m, and ring 3 spans 0–2000 m. The rings overlap — each outer ring includes the area of all inner rings. This is intentional: the **Intersection** step in Part 3 will clip each ring against block groups separately, and the `ringId` field will identify which ring each fragment belongs to.

5. Open the attribute table for `TRI_rings_3`. Confirm three rows with `ringId` values of 1, 2, and 3.

*[Figure: Three concentric buffer rings over the Philadelphia basemap, with the Layers panel showing TRI_rings_3 and its three ring features]*

---

## Part 3: Intersect buffer rings with block groups

The **Intersection** overlay clips block groups to each ring and retains both layers' attributes, including the `ringId` field that identifies which ring a fragment belongs to.

1. In the Processing Toolbox, search `intersection` and double-click **Intersection** (under **Vector overlay**).

2. Set parameters:

   | Parameter | Value |
   |---|---|
   | Input layer | `TRI_rings_3` |
   | Overlay layer | `phl_blockgroups_acs` |
   | Output | Save as `BG_intersect_rings.gpkg` |

3. Click **Run**. The output is a polygon layer where each feature is the portion of one block group that falls inside one buffer ring.

*[Figure: Block group fragments clipped to the three buffer rings; attribute table open alongside, showing ringId, GEOID, TOTPOP, and AREASQM columns]*

4. Open the attribute table for `BG_intersect_rings`. Confirm that `ringId`, `GEOID`, `TOTPOP`, `NH_WHITE`, `NH_BLACK`, `HISPANIC`, and `AREASQM` are all present.

---

## Part 4: Areal weighting — apportion population to each ring fragment

Block groups cross ring boundaries. You cannot assign an entire block group's population to a ring that only covers part of it. **Areal weighting** assumes population is distributed uniformly within each block group and apportions population in proportion to the share of the block group's area that fell inside the ring.

> **Public health note:** The areal-weighting assumption — that population is evenly distributed within a block group — is almost never literally true. A park occupying half a block group carries no residents but receives half the weight. This is one expression of the **Modifiable Areal Unit Problem (MAUP)**: results depend on how zone boundaries are drawn and on the granularity of the source zones. Use block groups rather than tracts when possible (smaller zones reduce error), and treat apportioned totals as estimates, not census counts.

**Step A — Calculate fragment area**

The intersection operation retains the original `AREASQM` value from the source block group, not the fragment's actual area. You need the fragment area.

1. Open the attribute table for `BG_intersect_rings`. Click the **Field Calculator** button (the abacus icon, or **Ctrl+I**).

2. Check **Create a new field**. Set:

   | Setting | Value |
   |---|---|
   | Output field name | `FRAG_SQMT` |
   | Output field type | Decimal number (double) |
   | Expression | `$area` |

   Click **OK**. Each row now holds the area of the ring fragment in square meters, calculated in the layer's CRS (EPSG:26918, units meters).

**Step B — Calculate the area fraction**

3. Open the Field Calculator again. Create a new field:

   | Setting | Value |
   |---|---|
   | Output field name | `AREA_FRAC` |
   | Output field type | Decimal number (double) |
   | Expression | `"FRAG_SQMT" / "AREASQM"` |

   Click **OK**. Values should be between 0 and 1. If any values exceed 1.0, the source `AREASQM` field may be in different units — recalculate `AREASQM` on the original `phl_blockgroups_acs` layer using `$area` in the Field Calculator, re-run the Intersection, and redo Steps A–B.

**Step C — Apportion population fields**

4. Use the Field Calculator to create four new Decimal (double) fields using the expressions below:

   | New field | Expression |
   |---|---|
   | `EST_TOTPOP` | `"TOTPOP" * "AREA_FRAC"` |
   | `EST_WHITE` | `"NH_WHITE" * "AREA_FRAC"` |
   | `EST_BLACK` | `"NH_BLACK" * "AREA_FRAC"` |
   | `EST_HISP` | `"HISPANIC" * "AREA_FRAC"` |

5. Toggle editing off: click the pencil icon at the top of the attribute table and choose **Save**.

*[Figure: Attribute table of BG_intersect_rings showing AREA_FRAC column with values between 0 and 1, and the four EST_ columns with decimal population estimates]*

---

## Part 5: Select block groups within 1 km — the "inside" set

For the demographic comparison you need two groups: block groups that intersect the 0–1 km zone, and those entirely outside it. You will use **Extract/Select by location** to identify the inside set.

1. First, select only the 0–1 km rings from `TRI_rings_3`. Open its attribute table and click **Select features using an expression** (the yellow epsilon icon). Enter the expression:

   ```
   "ringId" <= 2
   ```

   Click **Select Features**. Rings 1 and 2 (covering 0–500 m and 0–1000 m) are now highlighted. Close the table.

2. In the Processing Toolbox, search `select by location` and double-click **Select by location** (under **Vector selection**).

3. Set parameters:

   | Parameter | Value |
   |---|---|
   | Select features from | `phl_blockgroups_acs` |
   | Where the features | intersect |
   | By comparing to the features from | `TRI_rings_3` |
   | Only selected features | Checked |
   | Modify current selection by | creating new selection |

4. Click **Run**. Block groups that intersect the 0–1 km zone are now selected (highlighted in yellow on the map).

5. Right-click `phl_blockgroups_acs` in the **Layers** panel → **Export ▸ Save Selected Features As…**. Save as `BG_inside_1km.gpkg`. Make sure **Save only selected features** is checked.

6. Clear the selection on both layers: **Edit ▸ Select ▸ Deselect Features from All Layers** (or click the deselect button on the attribute toolbar).

---

## Part 6: Create the "outside 1 km" set using Difference

**Difference** in QGIS is the exact equivalent of Erase in ArcGIS Pro: it subtracts the overlay layer's geometry from the input, returning only what is left over.

1. In the Processing Toolbox, search `difference` and double-click **Difference** (under **Vector overlay**).

2. Set parameters:

   | Parameter | Value |
   |---|---|
   | Input layer | `phl_blockgroups_acs` |
   | Overlay layer | `BG_inside_1km` |
   | Output | Save as `BG_outside_1km.gpkg` |

3. Click **Run**. The output contains the block groups that had no intersection with the 0–1 km zone.

*[Figure: Philadelphia block groups with BG_inside_1km shown in blue and BG_outside_1km shown in gray, with the buffer rings overlaid]*

> **Note:** Difference removes full block-group polygons, not ring-clipped fragments. For the comparison table this is intentional — you want full demographic totals for each category, not areal-weighted portions.

---

## Part 7: Compare demographics inside versus outside 1 km

You will sum the population fields for each group using **Join attributes by location (summary)**. Alternatively, you can use **Basic statistics for fields** twice (once per layer) and record the results manually.

**Option A — Join attributes by location (summary)**

1. In the Processing Toolbox, search `join attributes by location summary` and open the tool.

2. Run once for the inside set:

   | Parameter | Value |
   |---|---|
   | Base layer | (create a single-feature layer, or use any reference polygon — see Note below) |
   | Join layer | `BG_inside_1km` |
   | Geometric predicate | intersects |
   | Fields to summarize | `TOTPOP`, `NH_WHITE`, `NH_BLACK`, `HISPANIC` |
   | Summaries to calculate | sum |
   | Output | `Stats_inside_1km.gpkg` |

   > **Note:** For a city-wide sum without a spatial predicate, use **Basic statistics for fields** (Option B) — it is simpler here.

**Option B — Basic statistics for fields (recommended)**

1. In the Processing Toolbox, search `basic statistics` and open **Basic statistics for fields**.

2. Run for the inside set:

   | Parameter | Value |
   |---|---|
   | Input layer | `BG_inside_1km` |
   | Field to calculate statistics on | `TOTPOP` (repeat for `NH_WHITE`, `NH_BLACK`, `HISPANIC`) |

   Record the **Sum** value from the results panel for each field. Repeat with `BG_outside_1km`.

3. From the sums you recorded, calculate percentages and complete this table in your Word document:

   | Group | Inside 1 km (n) | Inside 1 km (%) | Outside 1 km (n) | Outside 1 km (%) |
   |---|---|---|---|---|
   | Total population | | | | |
   | Non-Hispanic White | | | | |
   | Non-Hispanic Black | | | | |
   | Hispanic | | | | |
   | All other NH | | | | |

   Calculate "All other NH" as Total minus the three named groups. Calculate each percentage as (group n / Total n) × 100.

> **Public health note:** A higher share of non-White or Hispanic residents near TRI facilities compared to the rest of the city would be consistent with environmental justice research documenting disproportionate facility siting in communities of color. However, this cross-sectional proximity analysis cannot establish causation, cannot confirm actual chemical exposure, and does not account for historical patterns, zoning, or housing markets. The MAUP also applies: using census tracts instead of block groups would give different numbers.

---

## Part 8: Symbolize and map the buffer rings

1. In the **Layers** panel, arrange layers in this order from top to bottom: `phl_TRI_facilities`, `TRI_rings_3`, `phl_blockgroups_acs`.

2. Symbolize `TRI_rings_3`: right-click → **Properties ▸ Symbology**. Change the render type to **Categorized**, set the column to `ringId`, and click **Classify**. Assign distinct fill colors — for example, dark red for ring 1 (0–500 m), orange for ring 2 (500–1000 m), yellow for ring 3 (1000–2000 m). Set each fill opacity to 40% so block groups show through. Click **OK**.

3. Add a percent non-White field to `phl_blockgroups_acs`. Open its attribute table, click the **Field Calculator**, and create a new Decimal field:

   | Setting | Value |
   |---|---|
   | Output field name | `PCT_NONWHITE` |
   | Expression | `("TOTPOP" - "NH_WHITE") / "TOTPOP" * 100` |

4. Symbolize `phl_blockgroups_acs`: **Properties ▸ Symbology** → render type **Graduated**, column `PCT_NONWHITE`, color ramp **Blues** (light to dark), 5 classes, mode **Natural Breaks (Jenks)**. Click **Classify**, then **OK**.

5. Symbolize `phl_TRI_facilities`: **Properties ▸ Symbology** → **Single Symbol** → marker size 6, color black. Click **OK**.

6. Open the Print Layout: **Project ▸ New Print Layout**. Name it `Lab03b_map`.

7. In the layout, add: **Add Item ▸ Add Map** (drag to fill the page). Then add: **Add Item ▸ Add Legend**, **Add Scale Bar**, **Add North Arrow**, and **Add Label** (for a title and a data credit line crediting EPA TRI and ACS 2019–2023).

*[Figure: Print Layout showing the completed map with block groups shaded by percent non-White, three concentric TRI buffer rings, facility points, legend, scale bar, and north arrow]*

8. Export the layout: **Layout ▸ Export as PDF…**. Name the file `Lab03b_TRI_map.pdf`.

---

## Deliverables

Submit the following:

1. **Map PDF** (`Lab03b_TRI_map.pdf`): the buffer ring map from Part 8.
2. **QGIS project file** (`Lab03b_TRI_Buffer.qgz`): your saved project with all layers and symbology intact.
3. **Comparison table**: the completed inside/outside 1 km demographic table from Part 7, pasted into a Word document or submitted as a screenshot.
4. **Written interpretation (100 words)**: describe what the comparison table shows. Address at least two of the following: (a) direction and magnitude of any racial/ethnic difference; (b) the areal-weighting assumption and why it matters; (c) why proximity does not equal exposure; (d) one alternative analysis that would strengthen the inference.
5. **AI-corrected Methods paragraph** (see "Using AI in this lab"): the AI draft plus your tracked-changes corrections.

Label any AI-assisted output with the notation `[AI-assisted, reviewed by <your name>]`.

---

## Using AI in this lab

**Bounded task — draft a Methods paragraph, then fix it.**

After completing Part 7 and filling in your comparison table, paste the table values into Claude (claude.ai) or another AI assistant with this prompt:

> "I ran a buffer-and-overlay analysis in QGIS to compare racial/ethnic composition inside versus outside 1 km of EPA Toxic Release Inventory facilities in Philadelphia. I used multi-ring buffers, intersection with census block groups, areal weighting to apportion population by fragment area, and Basic statistics for fields to sum each group. Here are my summary counts: [paste your table]. Write a 150-word Methods paragraph suitable for a community health report, describing what I did and what the numbers show."

Read the AI draft carefully. It will likely make one or more of the following errors:

- Describe the areal-weighting step as if it produces exact population counts rather than estimates
- Omit or minimize the MAUP limitation
- Imply that living near a TRI facility means residents are exposed to toxic chemicals
- Overstate the strength of the comparison or imply a causal relationship

**Your task:** Paste the AI draft into a Word document, then use **Track Changes** to correct every error you can identify. Write a one-sentence note after each correction explaining why it was wrong.

**Guardrail — AI does not make analytic decisions in this lab.** The AI helped you draft a narrative. It did not choose the buffer distances, select the datasets, decide which demographic groups to compare, or interpret any statistical difference. Those decisions belong to you as the analyst. Never paste identifiable health data or patient addresses into a public AI tool.

---

## ArcGIS equivalents

The ArcGIS Pro and ArcGIS Online versions of this lab cover the same public health question and produce the same deliverables on the Esri platform.

| QGIS tool / step | ArcGIS Pro equivalent |
|---|---|
| **Multi-ring buffer (constant distance)** | **Multiple Ring Buffer** (Analysis Tools → Proximity) |
| **Intersection** | **Intersect** (Analysis Tools → Overlay) |
| **Select by location** | **Select By Location** (Map tab → Selection group) |
| **Difference** | **Erase** (Analysis Tools → Overlay) |
| **Basic statistics for fields** | **Summary Statistics** (Analysis Tools → Statistics) |
| Field Calculator, `$area` expression | **Calculate Geometry** → Area (Square Meters) |
| Field Calculator, `"field"` syntax | Field Calculator, `!field!` Python syntax |
| **Properties ▸ Symbology ▸ Graduated** | **Symbology** pane → Graduated Colors |
| **Project ▸ New Print Layout** | **Insert ▸ New Layout** (Insert tab) |

QGIS performs all the same operations and produces equivalent results. The **Difference** tool in QGIS is the exact equivalent of **Erase** in ArcGIS Pro. Print layouts in QGIS are fully featured and comparable to ArcGIS Pro layouts.

---

## Check your understanding

1. You open `phl_TRI_facilities.gpkg` and the CRS shows WGS 84 (EPSG:4326). You run **Multi-ring buffer (constant distance)** with a ring distance of 500 and produce three rings. What is wrong with the result, and what must you do before re-running the buffer?

2. After running **Intersection** between the buffer rings and the block groups, a classmate says: "Great — now we just sum `TOTPOP` for all fragments inside ring 1 to get the total population within 500 m." What is wrong with this approach, and what should you do instead?

3. The areal-weighting method assumes population is uniformly distributed within each block group. Describe one real-world situation in Philadelphia where this assumption would lead to a large overestimate of population within the 0–500 m ring.

4. A classmate argues: "Our table shows that 60% of people within 1 km of TRI facilities are non-White, but only 38% of people outside 1 km are non-White. This proves those facilities cause harm to communities of color." Identify two specific methodological problems with this causal claim.

5. In Part 6, you used **Difference** to create the "outside 1 km" set from full block-group polygons, not areal-weighted fragments. How would your summary results change if you had instead summed areal-weighted populations from `BG_intersect_rings` for fragments outside all rings? Which approach is more appropriate for the comparison table, and why?

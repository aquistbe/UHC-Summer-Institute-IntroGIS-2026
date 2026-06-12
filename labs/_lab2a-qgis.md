## Public health question

Philadelphia's park system is a critical health resource, but proximity depends entirely on how you measure distance. This lab asks: what share of Philadelphia children live within a 0.5-mile walk of a public park? The answer differs dramatically depending on the coordinate system used to run the buffer. One result is valid; the other is geographic nonsense. Learning to identify and fix this error before it reaches analysis is a foundational GIS skill.

---

## What you will learn

- Distinguish geographic coordinate systems (GCS) from projected coordinate systems (PCS)
- Check a layer's CRS in **Layer Properties ▸ Information** and read the project CRS button at the bottom-right of the QGIS window
- Interpret EPSG codes and look up unfamiliar CRS definitions at epsg.io
- Understand that QGIS reprojects layers on the fly for display — and why that is not enough for distance analysis
- Permanently reproject a layer using **Export ▸ Save Features As…** or the Processing **Reproject Layer** tool
- Run a **Buffer** operation in both the wrong CRS (degrees) and the correct CRS (feet) and see the difference on the map
- Use the **Measure** tool to document distance distortion across coordinate systems
- Use **Join attributes by location (summary)** and the **Field Calculator** to estimate children with park access by census tract
- Apply graduated symbology to display park access by poverty quintile
- Build a two-panel **Print Layout** comparing the wrong and correct buffers

---

## What you will produce

1. **Two-panel Print Layout PDF** (`Lab02a_buffer_comparison.pdf`) showing the same city at the same scale — left panel: 0.5-degree WGS84 buffer (wrong); right panel: 2,640-foot State Plane buffer (correct) — with labeled titles, your name, a scale bar, and a north arrow.
2. **A summary table** showing the percentage of children with park access (within the correct 0.5-mile buffer) by census-tract poverty quintile (five groups, lowest to highest poverty).
3. **A 100-word written interpretation** comparing access equity across poverty quintiles and explaining why the WGS84 buffer is invalid for this analysis. Label any AI-assisted portions.
4. **Your saved QGIS project file** (`Lab02a_Projections.qgz`).

---

## Data

All data are instructor-provided. Save them to your working folder before starting.

| Layer name | Description | Source |
|---|---|---|
| `PPR_properties.shp` | Philadelphia parks and recreation properties (polygons) | OpenDataPhilly |
| `phl_tracts_acs.shp` | Philadelphia census tracts with ACS child population (`CHILD_POP`, field B09001) and poverty rate (`POV_RATE`); includes `POV_QUINTILE` (1 = lowest, 5 = highest poverty) | NHGIS / Census Bureau |

> **Note:** Both layers are delivered in WGS84 (EPSG:4326). Your instructor will confirm exact field names for your data vintage. If a full-tract area field (`AREA_FULL`) is not present, you will calculate it in Part 4.

---

## Before you begin

**QGIS is free and open-source.** No license, credits, or organizational account required. Download QGIS 3.34 LTR from qgis.org if it is not already installed.

**Prerequisites:** Completion of Lab 1 (opening a project, adding data, basic navigation). No prior knowledge of projections is assumed — this lab teaches that.

**Plugins needed:** None for this lab. The tools used are all native to QGIS 3.34.

**Time estimate:** 2.5 to 3 hours.

**Have ready before starting:**

- QGIS 3.34 LTR open
- Your working folder with `PPR_properties.shp` and `phl_tracts_acs.shp`
- A web browser for epsg.io lookups

---

## Part 1: Create the project and inspect layer coordinate systems

### Set up the project

1. Open QGIS 3.34. Go to **Project ▸ New** (Ctrl+N) to create a blank project.
2. Go to **Project ▸ Save As…** Name the file `Lab02a_Projections` and save it to your working folder. QGIS creates a `.qgz` file.
3. Add your data: go to **Layer ▸ Add Layer ▸ Add Vector Layer**, click the **…** button, navigate to your working folder, and select `PPR_properties.shp`. Click **Add**, then repeat for `phl_tracts_acs.shp`. Alternatively, drag both files from the **Browser** panel directly onto the map canvas.

*[Figure: Layers panel showing PPR_properties and phl_tracts_acs loaded; both visible on the map canvas]*

### Check each layer's CRS

4. In the **Layers** panel, right-click `phl_tracts_acs` and select **Properties**. Click the **Information** tab (or the **Source** tab in some builds). Look at the **Coordinate Reference System** line.
5. You should see **EPSG:4326 — WGS 84**. This is a geographic coordinate system — location stored in degrees (longitude/latitude), not feet or meters.
6. Click **OK**. Repeat for `PPR_properties`. Both layers should show EPSG:4326.

*[Figure: Layer Properties ▸ Information tab showing EPSG:4326 WGS 84 for phl_tracts_acs]*

7. Look at the bottom-right corner of the QGIS window. You will see the project CRS button displaying **EPSG:4326**. QGIS automatically adopted the CRS of the first layer loaded. You can click this button at any time to open **Project ▸ Properties ▸ CRS**.

> **Note:** The **project CRS** and a **layer's stored CRS** are separate. QGIS reprojects layers on the fly for display so they line up on screen, but the coordinates stored on disk are unchanged. For any analysis involving distance or area, you must work in a projected CRS with linear units — not degrees. This is a critical distinction that ArcGIS Online hides from you by computing geodesic distances automatically; QGIS does not do this for you.

### Look up CRS codes on epsg.io

8. Open a browser and go to **epsg.io**. Search each code below and confirm the listed units:

| EPSG | Name | Units | Use for Philadelphia? |
|---|---|---|---|
| 4326 | WGS 84 | degrees | Display only — never for distance/area |
| 2272 | NAD83 / Pennsylvania South | US survey foot | Yes — recommended for this lab |
| 26918 | NAD83 / UTM Zone 18N | metre | Yes — also acceptable |

> **Public health note:** Any analysis involving distance or area — buffers, service areas, density calculations — requires a projected CRS with linear units (feet or meters). EPSG:4326 measures in degrees. Passing degrees to a distance tool produces meaningless results.

---

## Part 2: Run a buffer in the wrong coordinate system (WGS84)

This part deliberately does the analysis wrong so you can see the consequence.

### Confirm the project CRS is WGS84

1. Click the CRS button at the bottom-right of the window. Confirm it reads **EPSG:4326 — WGS 84**. If not, click **Project ▸ Properties ▸ CRS**, type `4326` in the filter box, select **WGS 84**, and click **OK**.

### Run the Buffer in degrees

2. Open the **Processing Toolbox** (**Processing ▸ Toolbox**, or Ctrl+Alt+T). In the search box at the top, type `buffer`. Double-click **Buffer** (under Vector geometry).
3. Set the parameters:

| Parameter | Value |
|---|---|
| Input layer | `PPR_properties` |
| Distance | `0.5` |
| Segments | 5 |
| Dissolve result | Checked |
| Output | Save to file — name it `parks_buffer_WGS84.gpkg` in your working folder |

4. Click **Run**.

> **Warning:** A 0.5 buffer run on WGS84 data is measured in **degrees**, not miles. At Philadelphia's latitude (~40° N), 1 degree of longitude is approximately 53 miles. A buffer of 0.5 degrees is therefore roughly **26 miles wide** — not half a mile. The result covers a region far larger than the entire city. You should never use degree-unit buffers for distance analysis. This step exists only to make that error visible and unforgettable.

5. When the tool finishes, `parks_buffer_WGS84` appears in the Layers panel. Note how the buffer polygons dwarf the city. Philadelphia is almost entirely contained within a single buffer zone.

*[Figure: WGS84 0.5-degree buffer covering a region far larger than Philadelphia — the "wrong" buffer]*

### Use the Measure tool to document the distortion

6. On the toolbar, click the **Measure Line** tool (ruler icon), or go to **View ▸ Measure ▸ Measure Line**. In the measure window that appears, set units to **Miles**.
7. Click one edge of a buffer polygon, then double-click the opposite edge. Note the approximate diameter in miles. It will be close to 53 miles — confirming that "0.5 degrees" at this latitude is nowhere near 0.5 miles.
8. Right-click to stop measuring. Write this value down; you will compare it to the correct buffer in Part 3.

---

## Part 3: Reproject the data and run the buffer correctly

### Permanently reproject to PA State Plane South (EPSG:2272)

1. In the **Layers** panel, right-click `PPR_properties` and select **Export ▸ Save Features As…**

> **Note:** This is how QGIS permanently reprojects a layer — it writes a new file in the target CRS. The original file on disk is unchanged. Do not confuse this with changing the project CRS (which only affects display) or with **Layer ▸ Set Layer CRS** (which relabels a layer without converting any coordinates — analogous to ArcGIS Pro's "Define Projection" mistake).

2. In the **Save Vector Layer as…** dialog, set:

| Parameter | Value |
|---|---|
| Format | GeoPackage |
| File name | `parks_StatePlane.gpkg` (in your working folder) |
| CRS | Click the globe icon; type `2272` in the filter; select **NAD83 / Pennsylvania South** (EPSG:2272); click **OK** |
| Add saved file to map | Checked |

3. Click **OK**. `parks_StatePlane` appears in the Layers panel.
4. Repeat for `phl_tracts_acs`, saving as `tracts_StatePlane.gpkg` with CRS EPSG:2272.

*[Figure: Save Features As dialog with EPSG:2272 selected as the target CRS]*

### Verify the reprojection

5. Right-click `parks_StatePlane` ▸ **Properties ▸ Information**. Confirm the CRS now shows **EPSG:2272 — NAD83 / Pennsylvania South**.
6. Watch the coordinate readout at the bottom-center of the QGIS window as you move your cursor over `parks_StatePlane`. Values should now be in the hundreds of thousands (US survey feet from the State Plane origin), not decimal degrees.

### Set the project CRS to EPSG:2272

7. Click the CRS button at the bottom-right of the window (or **Project ▸ Properties ▸ CRS**). Type `2272` in the filter, select **NAD83 / Pennsylvania South**, and click **OK**. The project now displays in State Plane feet.

### Run the Buffer in feet

8. In the **Processing Toolbox**, search `buffer` and open **Buffer** again. Set:

| Parameter | Value |
|---|---|
| Input layer | `parks_StatePlane` |
| Distance | `2640` |
| Segments | 5 |
| Dissolve result | Checked |
| Output | `parks_buffer_StatePlane.gpkg` in your working folder |

2,640 feet = 0.5 miles exactly. One mile = 5,280 feet; half a mile = 2,640 feet.

9. Click **Run**.

*[Figure: NAD83 PA State Plane buffer at 2,640 feet — sensible half-mile rings around Philadelphia parks]*

### Measure and compare

10. Use the **Measure Line** tool again (units: Miles). Measure across a buffer ring. The diameter should be approximately 1 mile (0.5 miles each side). Record this value alongside the WGS84 measurement from Part 2.

---

## Part 4: Compare the two buffers in a Print Layout

1. Go to **Project ▸ New Print Layout**. Name it `Buffer_Comparison` and click **OK**. The Print Layout designer opens.
2. Set page size to **Letter Landscape**: in the **Layout** panel on the right, under **Page**, set **Orientation** to **Landscape**.
3. Add the first map frame: **Add Item ▸ Add Map**. Draw a rectangle covering roughly the left half of the page. In the **Item Properties** panel, set the **CRS** to EPSG:4326 (WGS 84) by clicking the globe icon. This frame will show the WGS84 buffer. Set a fixed scale of approximately **1:500,000** so the enormous degree-unit buffer is visible.
4. In the Layers panel (back in the main QGIS window), turn on `parks_buffer_WGS84` and `phl_tracts_acs`; turn off the State Plane layers. Return to the Print Layout and click **Set Map Extent to Match Main Canvas** in the Item Properties if needed.
5. Add a second map frame covering the right half of the page. Set its CRS to EPSG:2272. Set the same scale (~1:100,000 works for the city extent). This frame will show the correct State Plane buffer.
6. For each map frame, use **Add Item ▸ Add Label** to add titles: "WGS84 Buffer (Wrong — 0.5 degrees)" and "State Plane Buffer (Correct — 2,640 feet / 0.5 miles)."
7. Add **Add Item ▸ Add Scale Bar** and **Add Item ▸ Add North Arrow** to each map frame. Add a title label with your name and date.
8. Export: **Layout ▸ Export as PDF…** Save as `Lab02a_buffer_comparison.pdf` in your working folder.

*[Figure: Finished Print Layout — left panel shows the enormous WGS84 degree buffer; right panel shows the correctly-sized half-mile State Plane buffer]*

---

## Part 5: Estimate children with park access by poverty quintile

### Prepare tract areas for areal interpolation

1. Switch back to the main QGIS map canvas. Make sure `tracts_StatePlane` is loaded and the project CRS is EPSG:2272.
2. Open the **Attribute Table** for `tracts_StatePlane` (right-click ▸ **Open Attribute Table**, or F6). Check whether a full-tract area field (`AREA_FULL`) already exists. If not, proceed to step 3. If it does, skip to step 5.
3. Toggle editing on (pencil icon at the top of the Attribute Table). Click the **Field Calculator** (abacus icon). Create a new field:

| Setting | Value |
|---|---|
| Create new field | Checked |
| Output field name | `AREA_FULL` |
| Output field type | Decimal number (real) |
| Expression | `$area` |

`$area` returns the polygon area in the units of the layer's CRS — here, square US survey feet. Click **OK**.

4. Toggle editing off and save changes.

### Intersect the correct buffer with census tracts

5. In the **Processing Toolbox**, search `intersection` and open **Intersection** (Vector overlay). Set:

| Parameter | Value |
|---|---|
| Input layer | `parks_buffer_StatePlane` |
| Overlay layer | `tracts_StatePlane` |
| Output | `buffer_tracts_intersect.gpkg` in your working folder |

6. Click **Run**. The output contains the portions of census tracts that fall inside the park buffer, carrying all original tract attributes.

> **Public health note:** This intersection clips tract polygons at the buffer boundary. For tracts only partially inside the buffer, you will estimate child population by assuming it is distributed uniformly across the tract's area — a method called areal interpolation. This is an approximation. It is imperfect but standard practice when individual-level data are unavailable.

### Calculate clipped area and estimated children in the buffer

7. Open the **Attribute Table** for `buffer_tracts_intersect`. Toggle editing on. Use the **Field Calculator** to create a new field:

| Setting | Value |
|---|---|
| Output field name | `AREA_CLIP` |
| Output field type | Decimal number (real) |
| Expression | `$area` |

8. Create another new field to estimate children inside the buffer:

| Setting | Value |
|---|---|
| Output field name | `CHILDREN_IN_BUF` |
| Output field type | Decimal number (real) |
| Expression | `"CHILD_POP" * ("AREA_CLIP" / "AREA_FULL")` |

This apportions each tract's child population proportional to the share of that tract's area inside the buffer.

9. Toggle editing off and save.

### Summarize by poverty quintile

10. In the **Processing Toolbox**, search `statistics by categories` and open **Statistics by categories** (Vector analysis). Set:

| Parameter | Value |
|---|---|
| Input vector layer | `buffer_tracts_intersect` |
| Field to calculate statistics on | `CHILDREN_IN_BUF` |
| Field(s) with categories | `POV_QUINTILE` |
| Output | `quintile_buffer_summary.gpkg` |

11. Click **Run**. Open the output table and note the `sum` column — this is estimated children in the buffer for each poverty quintile.
12. Repeat **Statistics by categories** on `tracts_StatePlane`, field `CHILD_POP`, category `POV_QUINTILE` to get total children per quintile citywide.
13. Open a spreadsheet or use the QGIS **Field Calculator** to compute percent access: divide each quintile's buffer sum by its citywide total and multiply by 100. Fill in the table below.

| Poverty quintile | Total children in city | Children in park buffer | % with park access |
|---|---|---|---|
| 1 (lowest poverty) | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 (highest poverty) | | | |

### Apply graduated symbology to visualize access by poverty quintile

14. Right-click `tracts_StatePlane` ▸ **Properties ▸ Symbology**. Change the render type from **Single Symbol** to **Graduated**.
15. Set **Value** to `POV_QUINTILE`, **Classes** to 5, **Mode** to **Equal Interval**, and choose a **ColorBrewer** sequential ramp (e.g., **YlOrRd** or **Blues**). Click **Classify**, then **OK**.
16. This map shows poverty quintile distribution. Consider adding a second map showing `CHILDREN_IN_BUF` as a graduated layer on `buffer_tracts_intersect` to visualize where children with park access live.

*[Figure: Graduated symbology on census tracts by POV_QUINTILE, with park buffer overlay]*

---

## Deliverables

Submit the following as one PDF or as directed by your instructor:

1. **Side-by-side buffer comparison PDF** (`Lab02a_buffer_comparison.pdf`) — two map panels at equal scale showing the WGS84 buffer (wrong) and the State Plane buffer (correct), with labeled titles, scale bars, north arrows, and your name.
2. **Completed access table** — the poverty quintile table from Part 5 with all five rows filled in. Screenshot or paste into your submission PDF.
3. **100-word written interpretation** answering: Which quintile has the highest park access rate? The lowest? What does this pattern suggest about park equity in Philadelphia? Why is the WGS84 buffer invalid for this analysis? Label any AI-assisted portions.
4. **Your saved project file** `Lab02a_Projections.qgz`.

---

## Using AI in this lab

**Bounded AI task — interrogating a CRS definition:**

1. In QGIS, right-click any layer ▸ **Properties ▸ Information**. Scroll to the CRS section and copy the full CRS text block, including the PROJ string or WKT definition.
2. Paste it into Claude (claude.ai) with this prompt: "Here is a coordinate reference system definition. Is this geographic or projected? What are its linear units? Is it appropriate for measuring distances in Philadelphia, PA? Explain why or why not."
3. Read the response critically. Does the AI correctly identify the units? Compare its answer to what you found at epsg.io. AI explains CRS text well — treat it as a scaffold, not an authority.

You can also ask Claude: "Here is a QGIS expression I wrote to calculate children in the buffer: `"CHILD_POP" * ("AREA_CLIP" / "AREA_FULL")`. Is the logic correct for areal interpolation? What assumptions does it make?" Then evaluate whether the explanation matches what you learned in lab.

**Guardrails:**

- AI helps with **explaining** (what does this CRS definition mean?) and **critiquing** (is my projection choice reasonable?). It does not make analytic decisions for you.
- AI does **not** select the right projection for your study area — you do.
- AI does **not** interpret your equity findings — that requires your public health judgment.
- **Never paste patient data, addresses, or any identifiable health information into a public AI tool.** A CRS string or QGIS expression contains no PHI and is safe to share.
- Label any AI-assisted text in your deliverables.

---

## ArcGIS equivalents

Students using ArcGIS Pro can complete this workflow with the following equivalents. Full Pro and ArcGIS Online versions of this lab exist for those platforms.

| QGIS operation | ArcGIS Pro equivalent |
|---|---|
| Layer Properties ▸ Information (CRS) | Layer Properties ▸ Source ▸ Spatial Reference |
| Project CRS button (bottom-right) | Map Properties ▸ Coordinate System |
| Project ▸ Properties ▸ CRS | Map Properties ▸ Coordinate System |
| Export ▸ Save Features As… (with target CRS) | Analysis ▸ Tools ▸ **Project** tool (Data Management) |
| Processing ▸ Reproject Layer | Same **Project** tool |
| Processing ▸ Buffer (native:buffer) | Analysis ▸ Tools ▸ **Buffer** |
| View ▸ Measure ▸ Measure Line | Map tab ▸ Inquiry ▸ **Measure** |
| Processing ▸ Intersection | Analysis ▸ Tools ▸ **Intersect** |
| Field Calculator (`$area`) | Field Calculator or Calculate Geometry |
| Processing ▸ Statistics by categories | Analysis ▸ Tools ▸ **Summary Statistics** |
| Project ▸ New Print Layout | Insert ▸ New Layout |

> **Key difference:** ArcGIS Online computes buffers geodesically by default, so degree-unit errors are less likely to surface there. QGIS and ArcGIS Pro both perform planar buffering by default — the CRS of your data determines whether the result is valid. This makes CRS management more visible, and more important, in desktop GIS.

---

## Check your understanding

1. A colleague sends you a shapefile of school locations in Philadelphia. You load it into QGIS and it displays correctly over an OpenStreetMap basemap. Does this confirm the layer's CRS is correct for distance analysis? Why or why not?

2. What is the difference between **Export ▸ Save Features As…** (with a new target CRS) and **Layer ▸ Set Layer CRS** in QGIS? Which one actually converts coordinates, and when would misusing the second one damage your data?

3. You are asked to create a 1-kilometer buffer around all FQHC clinic locations in Philadelphia. Which EPSG code would you choose — EPSG:4326, EPSG:2272, or EPSG:26918 — and why? Is there more than one acceptable answer?

4. In the Print Layout you produced, the WGS84 buffer polygon extends far beyond the city limits. Explain in plain language why a "0.5-degree" buffer at Philadelphia's latitude (~40° N) produces a result approximately 26 miles wide.

5. Your access table shows that census tracts in the highest-poverty quintile have a lower percentage of children with park access than tracts in the lowest-poverty quintile. List two alternative explanations for this pattern that do not involve park location — and identify what additional data you would need to evaluate each explanation.

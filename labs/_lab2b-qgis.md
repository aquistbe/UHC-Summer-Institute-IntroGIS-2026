## Public health question

A community health team is conducting a walking audit of four city blocks in West Philadelphia to assess pedestrian injury risk. Observations about sidewalk quality, crossing infrastructure, and traffic calming have been recorded on paper forms. To analyze the spatial distribution of hazardous conditions, those records must enter a GIS as georeferenced features. This lab teaches you to digitize point observations, street-segment lines, and a study-area polygon — and to record coded attributes — so the data can be mapped, queried, and linked to injury outcomes.

---

## What you will learn

- Add an imagery basemap using XYZ Tiles in the Browser panel
- Create GeoPackage layers (point, line, polygon) with defined fields via **Layer ▸ Create Layer ▸ New GeoPackage Layer**
- Configure **Value Map** widgets for coded audit categories via **Properties ▸ Attributes Form**
- Set up snapping (**Project ▸ Snapping Options**) and toggle editing with the pencil icon
- Digitize point, line, and polygon features; enter attributes in the form
- Edit vertices with the **Vertex Tool**
- Populate length, area, and coordinate fields using the **Field Calculator** with `$length`, `$area`, and `transform()` expressions in a projected CRS
- Export a **Print Layout** PDF

---

## What you will produce

1. A GeoPackage (`Lab02b_WalkingAudit.gpkg`) with three layers: `audit_sites` (points), `street_segments` (lines), `study_area` (polygon)
2. Fully populated attribute tables for all three layers
3. An exported Print Layout PDF showing all three layers over imagery

---

## Data

All data are created by you during the session. No pre-existing GIS files are required.

| Layer | Description | Source |
|---|---|---|
| OpenStreetMap (basemap) | Street reference | XYZ Tiles in the QGIS Browser panel |
| Google Hybrid (imagery) | Aerial imagery for digitizing | XYZ Tiles — new connection (URL in Part 2) |
| Walking-audit codebook | Coded attribute categories | This handout |

> **Note:** QGIS is free and open-source. There are no license fees, credits, or organizational account barriers.

### Walking-audit attribute codebook

| Attribute | Field name | Type | Coded values |
|---|---|---|---|
| Site ID | `site_id` | Integer | 1, 2, 3 … (sequential) |
| Lighting | `lighting` | Text (10) | `none` · `poor` · `adequate` |
| Sidewalk condition | `sidewalk` | Text (10) | `good` · `cracked` · `absent` |
| Crossing aids | `crossing` | Text (10) | `none` · `marked` · `signal` · `curb_ramp` |
| Traffic calming | `traffic_calm` | Text (15) | `none` · `speed_hump` · `bumpout` |
| Notes | `notes` | Text (100) | Free text |
| Segment ID | `seg_id` | Integer | Lines only |
| Length (ft) | `length_ft` | Double | Calculated — lines |
| Area (sq ft) | `area_sqft` | Double | Calculated — polygon |
| Latitude | `lat_dd` | Double | Calculated — points |
| Longitude | `lon_dd` | Double | Calculated — points |

> **Public health note:** Coded values prevent free-text inconsistencies — a common data quality problem when multiple field workers contribute records. Consistent coding is essential for counting exposures across a study area.

---

## Before you begin

**Prerequisites:** Lab 1 (QGIS orientation) and Lab 2a (projections).

**QGIS version:** QGIS 3.34 LTR — free at [qgis.org](https://qgis.org). No credits or licensing required.

**Plugins:** Optional — install **QuickMapServices** (**Plugins ▸ Manage and Install Plugins…**) for one-click basemap access.

**Estimated time:** 2 hours.

---

## Part 1: AI-assisted codebook design

1. Open Claude (claude.ai) or another approved AI assistant.
2. Paste this prompt exactly:

   > *"We're auditing 4 blocks in West Philly to study pedestrian injury risk. We need to capture sidewalk quality, crossing infrastructure, and traffic calming. Draft a simple field codebook — field name, description, and coded values — for a GIS attribute table."*

3. Compare the AI output to the codebook table above. Note matches, differences, and missing categories.
4. Write two sentences explaining one change you would make to the AI draft and why. Save as `codebook_notes.txt`.

> **Using AI in this lab:** The AI provides a starting scaffold. You, as the public health professional, decide which categories fit a pedestrian-injury study. The AI does not know your study population, IRB protocol, or local conditions. Always review and refine AI-generated protocols before research use.

---

## Part 2: Create a project and add imagery

1. **Project ▸ New**. Set the project CRS: click the CRS button at the bottom-right of the window, search `2272`, and select **NAD83 / Pennsylvania South (ftUS)**. Click **OK**.

   > **Public health note:** EPSG:2272 uses US feet. Setting this before digitizing means `$length` and `$area` in the Field Calculator return feet and square feet directly — matching your audit forms.

2. Save the project: **Project ▸ Save As…** → `Lab02b_WalkingAudit.qgz` in your `Lab02b` folder.
3. In the **Browser** panel, expand **XYZ Tiles** and double-click **OpenStreetMap** to add it.
4. Right-click **XYZ Tiles ▸ New Connection…**. Set **Name** to `Google Hybrid` and **URL** to `https://mt1.google.com/vt/lyrs=y&x={x}&y={y}&z={z}`. Click **OK**, then double-click the new connection to add imagery to the map.
5. Navigate to West Philadelphia — zoom to approximately 40th Street and Baltimore Avenue.

*[Figure: QGIS map canvas at ~1:5,000 showing West Philadelphia imagery with Layers panel visible]*

---

## Part 3: Create GeoPackage layers

GeoPackage (`.gpkg`) is QGIS's preferred open format — a single file that holds multiple layers, free of the shapefile's field-length limits.

### 3A. audit_sites (points)

1. **Layer ▸ Create Layer ▸ New GeoPackage Layer…**
2. Click **…** next to Database, navigate to `Lab02b`, name the file `Lab02b_WalkingAudit.gpkg`. **Table name:** `audit_sites`. **Geometry type:** Point. **CRS:** EPSG:2272.
3. Add each field using the **New Field** section and clicking **Add to Fields List**:

   | Field name | Type | Length |
   |---|---|---|
   | `site_id` | Integer | — |
   | `lighting` | Text (string) | 10 |
   | `sidewalk` | Text (string) | 10 |
   | `crossing` | Text (string) | 10 |
   | `traffic_calm` | Text (string) | 15 |
   | `notes` | Text (string) | 100 |
   | `lat_dd` | Decimal number | — |
   | `lon_dd` | Decimal number | — |

4. Click **OK**. The layer appears in the Layers panel.

**Configure Value Map widgets:** Right-click `audit_sites` → **Properties ▸ Attributes Form**. Select `lighting`; change Widget type to **Value Map**. Add rows for each coded value (`none`, `poor`, `adequate`). Repeat for `sidewalk`, `crossing`, and `traffic_calm`.

*[Figure: Attributes Form tab showing Value Map widget for the lighting field]*

### 3B. street_segments (lines)

1. **Layer ▸ Create Layer ▸ New GeoPackage Layer…** — select the existing `Lab02b_WalkingAudit.gpkg` as the database (do not create a new file). **Table name:** `street_segments`. **Geometry type:** Line String. **CRS:** EPSG:2272.
2. Add fields: `seg_id` (Integer), `lighting`/`sidewalk`/`crossing`/`traffic_calm` (Text, same lengths as above), `notes` (Text 100), `length_ft` (Decimal).
3. Configure Value Map widgets for the four coded fields.

### 3C. study_area (polygon)

1. Same process — **Table name:** `study_area`. **Geometry type:** Polygon. **CRS:** EPSG:2272.
2. Add one field: `area_sqft` (Decimal number). Click **OK**.

---

## Part 4: Configure snapping

1. **Project ▸ Snapping Options…** (or click the magnet icon on the Snapping toolbar).
2. Activate snapping (click the horseshoe magnet so it darkens). Set **Mode:** All Layers; enable **Vertex** and **Segment**; set tolerance to **10 feet**.
3. Enable **Topological Editing** so that moving a shared vertex updates all connected features.

> **Note:** 10 feet suits block-scale data. Too small causes missed connections; too large forces unintended snapping.

---

## Part 5: Digitize the study-area polygon

1. Select `study_area` in the Layers panel. Click the **pencil (Toggle Editing)** icon.
2. Set a semi-transparent fill so imagery remains visible: **Properties ▸ Symbology ▸ Simple Fill** → set **Opacity** to 50% in the Layer Rendering section.
3. Click **Add Polygon Feature**. Click the four-to-six street corners defining the study boundary. **Right-click** to finish. Leave `area_sqft` blank. Click **OK** in the attribute form.
4. **Save Layer Edits** (floppy disk icon).

**To adjust a vertex:** press V for the **Vertex Tool**, click a vertex (turns blue), drag. Double-click an edge to insert a new vertex.

---

## Part 6: Digitize street segments

1. Select `street_segments`, toggle editing on. Click **Add Line Feature**.
2. Left-click along one block face to place vertices. **Right-click** to finish. In the attribute form, enter `seg_id`, coded values for the four audit fields, and a note. Leave `length_ft` blank.
3. Digitize at least **four block faces**. The snap indicator confirms vertex coincidence at existing endpoints.
4. Mid-digitize: press **Backspace** to remove the last vertex; **Escape** to abandon the segment.
5. To edit a finished segment, use the **Vertex Tool** (V): click a vertex and drag; double-click an edge to insert a vertex.
6. Save layer edits.

*[Figure: Map showing four line segments along West Philadelphia block faces, with snap indicator visible at an intersection]*

---

## Part 7: Digitize audit observation points

1. Select `audit_sites`, toggle editing on. Click **Add Point Feature**.
2. Left-click a sidewalk location. In the attribute form, enter `site_id`, coded values, and notes. Leave `lat_dd`/`lon_dd` blank. Click **OK**.
3. Digitize at least **six points** distributed across the study area, mixing good and poor conditions.

> **Warning:** To remove a misplaced point immediately, press **Ctrl+Z** (Undo). To delete a saved point, activate **Select Features** (yellow arrow), click the point, then press **Delete**.

4. To move a point, use **Edit ▸ Move Feature(s)**: click the point, then click the corrected location.
5. Save layer edits. Then save the project (Ctrl+S).

> **Mobile collection note:** **QField** (qfield.cloud) syncs your `.qgz` project, GeoPackage layers, and Value Map widgets to a tablet or phone for GPS-enabled field data collection.

*[Figure: Map showing six point symbols over West Philadelphia imagery, attribute form open with value-map drop-downs visible]*

---

## Part 8: Field Calculator — lengths, areas, and coordinates

Open the attribute table (**F6**), click the **Field Calculator** (abacus icon), check **Update existing field**, select the target field, and enter the expression below.

| Layer | Field | Expression |
|---|---|---|
| `street_segments` | `length_ft` | `$length` |
| `study_area` | `area_sqft` | `$area` |
| `audit_sites` | `lat_dd` | `y(transform($geometry, 'EPSG:2272', 'EPSG:4326'))` |
| `audit_sites` | `lon_dd` | `x(transform($geometry, 'EPSG:2272', 'EPSG:4326'))` |

> **Note:** `$length` and `$area` return values in the layer CRS units (feet, because EPSG:2272 uses US feet). Plain `$y` would return northing in feet, not latitude — so `transform()` to WGS84 is needed for decimal-degree coordinates.

Verify: latitude ≈ 39.95, longitude ≈ −75.20. Values well outside that range indicate a CRS mismatch. Save all layer edits, then save the project (Ctrl+S).

---

## Part 9: Export a Print Layout

1. **Project ▸ New Print Layout…**, name it, click **OK**.
2. **Add Item ▸ Add Map** — draw a frame across most of the page.
3. Add: **Label** (title: *Walking Audit — West Philadelphia Study Area*), **Legend**, **Scale Bar** (feet), **North Arrow**, and a Label with your name and date.
4. **Layout ▸ Export as PDF…** → save as `Lab02b_map.pdf`.

---

## Deliverables

| File | Description |
|---|---|
| `Lab02b_WalkingAudit.gpkg` (zipped) | GeoPackage with all three layers and populated attributes |
| `Lab02b_WalkingAudit.qgz` | QGIS project file |
| `Lab02b_map.pdf` | Print Layout export showing all layers over imagery |
| `codebook_notes.txt` | Two-sentence reflection from Part 1; note AI origin and your revision |
| Short interpretation (100 words) | Which block face or intersection is most hazardous, and which two coded conditions drove that judgment? (Submit as a text block in the PDF or a separate `.txt`) |

---

## Using AI in this lab

**Permitted (Part 1):** Draft an audit codebook from a 3-sentence description, then evaluate and refine it as the subject-matter expert.

**Guardrails that always apply:**

- AI assists with **scaffolding**, **explaining**, and **critiquing** — not analytic decisions, feature selection, or interpretation of field observations.
- Never paste PHI or identifiable addresses into a public LLM. This lab uses illustrative data, but build the habit now.
- Label all AI-assisted output. In `codebook_notes.txt`, state the draft was AI-generated and describe your revision.
- You may ask AI to help write or debug a Field Calculator expression; always test the result on your own data.

---

## ArcGIS equivalents

The matching **ArcGIS Pro** version (Lab 2b Pro) and an **ArcGIS Online** version exist for those platforms.

| QGIS (this lab) | ArcGIS Pro equivalent |
|---|---|
| **Layer ▸ Create Layer ▸ New GeoPackage Layer** | Create Feature Class wizard (in a file geodatabase) |
| Pencil icon → Add Point / Line / Polygon Feature | **Edit** tab → **Create Features** panel |
| **Project ▸ Snapping Options** | **Edit** tab → Snapping group → Snapping toggle |
| **Vertex Tool** | **Modify Features** → Vertices |
| **Field Calculator** with `$length`, `$area`, `transform()` | Right-click column → **Calculate Geometry** |
| **Save Layer Edits** (floppy disk) | **Edit** tab → **Save** (Manage Edits) |
| **Project ▸ New Print Layout** | **Insert** tab → **New Layout** |
| GeoPackage (`.gpkg`) | File geodatabase (`.gdb`) |
| Value Map widget (Attributes Form) | Coded-value domain |

---

## Check your understanding

1. Why set the project CRS to EPSG:2272 before digitizing, rather than leaving it in WGS84?
2. What data quality problem could arise in a multi-auditor study if `sidewalk` is a plain text field rather than a Value Map widget?
3. List the exact steps to use the Vertex Tool in QGIS to move an incorrect vertex on a finished line segment.
4. A colleague's `street_segments` layer was accidentally created with CRS EPSG:4326. What units would `$length` return, and why would the values look wrong?
5. A colleague proposes skipping the study-area polygon and using the bounding extent of points and lines instead. What analytic problem arises when calculating the percentage of block faces with absent sidewalks?

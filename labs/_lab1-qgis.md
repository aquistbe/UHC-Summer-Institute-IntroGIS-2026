## Public health question

Where are the most heat-vulnerable neighborhoods in Philadelphia, and how do cooling resources and flood hazards align with them? Extreme heat is the leading weather-related cause of death in the United States, and urban heat exposure is unequally distributed. This lab uses Philadelphia neighborhood data to begin exploring that spatial inequality — asking whether the areas most vulnerable to heat are also the areas best served by cooling infrastructure such as pools and spraygrounds.

---

## What you will learn

- Install and launch QGIS 3.34 LTR; understand the `.qgz` project file format
- Add vector layers (shapefile) via the Data Source Manager and the Browser panel
- Add a free OpenStreetMap basemap using XYZ Tiles
- Arrange layers in the Layers panel; set relative paths for project portability
- Apply three symbology types: Single Symbol, Graduated choropleth, and Categorized
- Compare Equal Interval, Quantile, and Natural Breaks classification schemes
- Add labels; create and use spatial bookmarks; measure distances
- Build a full Print Layout (map frame, legend, title, north arrow, scale bar) and export to PDF

---

## What you will produce

Three PDF maps of Philadelphia exported from a single QGIS project (`.qgz`):

1. **Map 1 — Flood and green infrastructure:** GSI project locations over FEMA 100-year flood zones, with census tracts as a base.
2. **Map 2 — Heat vulnerability choropleth:** Tract-level heat vulnerability index with graduated colors; includes an Equal Interval vs. Quantile comparison.
3. **Map 3 — Cooling resources:** Heat vulnerability choropleth with pools and spraygrounds as distinct point symbols.

---

## Data

All files are instructor-provided. Keep them in the same folder as your `.qgz` project file to preserve relative paths.

| Layer name | Description | Source |
|---|---|---|
| `Philadelphia_census_tracts.shp` | 2020 census tract boundaries, Philadelphia County | TIGER/Line via OpenDataPhilly |
| `GSI_public_projects.shp` | Philadelphia Water Dept. public green stormwater infrastructure projects | OpenDataPhilly |
| `100_yr_flood.shp` | FEMA 100-year (1% annual chance) flood zone polygons | OpenDataPhilly |
| `heat_vulnerability_ct.shp` | Heat vulnerability index by census tract; field `HVI` = index score | OpenDataPhilly / Philadelphia Dept. of Public Health |
| `PPR_Spraygrounds.shp` | Philadelphia Parks and Recreation sprayground locations | OpenDataPhilly |
| `PPR_Swimming_Pools.shp` | Philadelphia Parks and Recreation public swimming pool locations | OpenDataPhilly |

> **Note:** QGIS stores layer references, not the data. Keep all files and the `.qgz` in the same folder; do not save to a Desktop or USB drive that may be unavailable at your next session.

---

## Before you begin

- **QGIS is free and open-source.** No license, organizational account, or credits are required. Download QGIS 3.34 LTR from [qgis.org](https://qgis.org) if not installed. On Windows use the standalone LTR installer; on macOS use the installer from qgis.org.
- **Plugin to install — QuickMapServices:** Go to **Plugins ▸ Manage and Install Plugins…**, search `QuickMapServices`, click **Install Plugin**. Then go to **Web ▸ QuickMapServices ▸ Settings ▸ More services ▸ Get contributed pack** to unlock the full basemap catalog.
- **Prerequisites:** Basic file navigation. No prior GIS experience required.
- **Estimated time:** 2.5–3 hours.

---

## Part 1: Create and configure a project

### 1.1 Launch QGIS and create a project

1. Open QGIS 3.34 LTR. On Windows: **Start ▸ OSGeo4W ▸ QGIS**. On macOS: Applications folder. Dismiss the tip dialog if it appears.
2. Click **Project ▸ New** (Ctrl+N). Immediately save: **Project ▸ Save As…**, navigate to your lab data folder, name it `Lab01_Philadelphia`, click **Save**. The title bar confirms the project name.
3. Set relative paths: **Project ▸ Properties… ▸ General**. Confirm **Save paths** is set to **Relative**. Click **OK**.

> **Note:** The `.qgz` is a recipe, not a data store. Relative paths let you move the entire project folder to another computer without broken layer links.

### 1.2 Set the project CRS

4. Click **Project ▸ Properties… ▸ CRS**. Search `2272`, select **NAD83 / Pennsylvania South (ftUS)** (EPSG:2272), click **OK**. The CRS indicator in the lower-right of the window updates. QGIS reprojects layers on the fly for display.

---

## Part 2: Add data and symbolize Map 1 — flood and green infrastructure

### 2.1 Add layers

1. Open **Layer ▸ Data Source Manager** (Ctrl+L). On the **Vector** tab, click **…** and select `Philadelphia_census_tracts.shp`, then **Add**. Repeat for `100_yr_flood.shp` and `GSI_public_projects.shp`. Click **Close**.

   You can also drag files from the **Browser** panel directly onto the map canvas.

2. In the **Layers** panel, drag layers to this order (top to bottom): `GSI_public_projects`, `100_yr_flood`, `Philadelphia_census_tracts`.

> **Public health note:** Layer drawing order controls visibility. Points and lines must sit above polygon layers or they will be hidden.

### 2.2 Add a basemap

3. In the **Browser** panel, expand **XYZ Tiles** and double-click **OpenStreetMap**. Drag the basemap layer below `Philadelphia_census_tracts` in the Layers panel. Click **View ▸ Zoom Full** (Ctrl+Shift+F) to zoom to all layers.
4. Save a bookmark: **View ▸ New Spatial Bookmark…** (Ctrl+B), name it `Philadelphia full extent`, click **Save**.

### 2.3 Symbolize the layers

**Census tracts — outline only:**

5. Double-click `Philadelphia_census_tracts` to open **Layer Properties ▸ Symbology** (Single Symbol). Click the symbol patch → **Simple Fill**. Set **Fill color** to **No Color** (opacity 0). Set **Stroke color** to medium gray, **Stroke width** `0.3 mm`. Click **OK ▸ Apply ▸ OK**.

**Flood zones — light blue fill:**

6. Double-click `100_yr_flood`. Set **Fill color** to light blue, ~50% opacity; **Stroke color** same blue, `0.2 mm`. Click **OK ▸ Apply ▸ OK**.

> **Public health note:** FEMA 100-year flood zones mark areas with a 1% annual flood probability. Repeated flooding causes displacement and lasting health harm — disproportionately in lower-income communities.

**GSI points — dark green:**

7. Double-click `GSI_public_projects`. Choose **Simple Marker**, **Fill color** dark green, **Size** `2.5 mm`. Click **OK ▸ Apply ▸ OK**.

*[Figure: Map canvas showing gray tract outlines, light blue flood zones, and dark green GSI points over the OSM basemap]*

---

## Part 3: Print Layout — Map 1

1. Click **Project ▸ New Print Layout…**, name it `Map1_GSI_FloodZones`, click **OK**.
2. Right-click the blank page → **Page Properties**. Set **Size** to **Letter** and **Orientation** to **Landscape**.
3. **Map frame:** Click **Add Item ▸ Add Map**, drag a rectangle on the page leaving margin for labels.
4. **Title:** **Add Item ▸ Add Label**. Type: `Green Stormwater Infrastructure and 100-Year Flood Zones, Philadelphia`. Set bold, 14 pt in the **Item Properties** panel.
5. **North Arrow:** **Add Item ▸ Add North Arrow**. Place in the lower-right margin.
6. **Scale Bar:** **Add Item ▸ Add Scale Bar**. In Item Properties set **Units** to Miles, adjust segments so the bar reads clearly (e.g., 0–2–4 miles).
7. **Legend:** **Add Item ▸ Add Legend**. Remove the basemap entry (select it, click −). Verify the three layers appear with legible labels.
8. Add a text label with your name and today's date.
9. Export: **Layout ▸ Export as PDF…**, save as `Map1_GSI_FloodZones.pdf`, **150 dpi** minimum. Return to the main window and save the project (Ctrl+S).

> **Warning:** Save frequently (Ctrl+S). Unsaved work is lost if QGIS crashes.

*[Figure: Completed Print Layout for Map 1 with title, legend, north arrow, and scale bar]*

---

## Part 4: Symbology for Map 2 — heat vulnerability choropleth

### 4.1 Add the layer and apply Graduated symbology

1. Add `heat_vulnerability_ct.shp` via the Data Source Manager. Drag it to the top of the Layers panel.

> **Public health note:** The heat vulnerability index (`HVI`) combines land surface temperature, impervious cover, tree canopy, poverty, and age to flag tracts with compounded risk during heat events. Every index reflects its authors' choices about which factors to include and how to weight them — limitations worth examining.

2. Double-click the layer → **Symbology**. Change the render type to **Graduated**. Configure:

| Parameter | Value |
|---|---|
| Value | `HVI` |
| Color ramp | YlOrRd (or similar yellow-to-red sequential ramp) |
| Classes | 5 |
| Mode | Quantile |

3. Click **Classify**, then **Apply**.

### 4.2 Compare classification schemes

4. Change **Mode** to **Equal Interval**, click **Classify** — note how class membership shifts. Try **Natural Breaks (Jenks)** as a third option.

> **Public health note:** Quantile places equal numbers of tracts in each class — readable but obscures absolute magnitude. Equal Interval preserves the true data range but often crowds most tracts into one or two classes for skewed indices. Natural Breaks minimizes within-class variance. Your choice is an analytic decision, not a default.

*[Figure: Side-by-side views of Equal Interval (left) vs. Quantile (right) for the same 5-class yellow-to-red scheme]*

5. Choose your final scheme. Click **Apply ▸ OK**.

### 4.3 Add labels and save a bookmark

6. In Layer Properties → **Labels** tab, set mode to **Single Labels**, choose `GEOID` as the label field, font size 6 pt. Click **Apply ▸ OK**. (Turn labels off at small scales to reduce clutter.)
7. Zoom to a cluster of high-HVI tracts (e.g., North or West Philadelphia). Save a bookmark: **View ▸ New Spatial Bookmark…**, name `High-vulnerability cluster`.

---

## Part 5: Measure tool

1. With the high-vulnerability cluster in view, click **View ▸ Measure ▸ Measure Line** (Ctrl+Shift+M on Windows; Cmd+Shift+M on macOS).
2. Click the centroid of a high-HVI tract, then click the nearest GSI point visible from Map 1. Double-click to finish.
3. In the Measure dialog, confirm units are set to **Miles**. Record the distance — you will report it in your deliverables.
4. Press **Escape** or close the dialog to exit the tool.

> **Public health note:** Straight-line distance is not walking or travel distance. A later lab covers network-based access. The Measure tool gives a rough proximity check only.

---

## Part 6: Map 3 — cooling resources

1. Add `PPR_Spraygrounds.shp` and `PPR_Swimming_Pools.shp`. Drag both to the top of the Layers panel.
2. Symbolize `PPR_Spraygrounds`: Single Symbol, **Simple Marker**, bright blue fill, **Size** `3 mm`.
3. Symbolize `PPR_Swimming_Pools`: Single Symbol, a **different shape** (e.g., square or diamond), teal or cyan fill, **Size** `3 mm`.

> **Note:** Two separate layers with distinct shapes and colors is cleaner than Categorized on a merged layer. The legend will show one entry per layer.

*[Figure: Heat vulnerability choropleth with blue circle sprayground symbols and teal square pool symbols]*

4. Create a new Print Layout: **Project ▸ New Print Layout…**, name `Map3_CoolingResources`. Add map frame, title (`Heat Vulnerability and Public Cooling Resources, Philadelphia`), legend (include choropleth classes and both point types with plain-language labels), north arrow, scale bar, and your name/date.
5. Export as `Map3_CoolingResources.pdf`, 150 dpi.
6. Also export a final Map 2 layout (`Map2_HeatVulnerability.pdf`) with title `Heat Vulnerability Index by Census Tract, Philadelphia`.
7. Save the project (Ctrl+S).

---

## Deliverables

Submit to your instructor by the deadline:

1. **Three PDF maps:** `Map1_GSI_FloodZones.pdf`, `Map2_HeatVulnerability.pdf`, `Map3_CoolingResources.pdf`
2. **Classification comparison note (100 words max):** Which scheme did you choose for Map 2, why, and what does it emphasize or obscure?
3. **Measured distance:** The straight-line distance from Part 5 (in miles) and a brief description of the two endpoints.
4. **AI-assisted explanation (labeled):** The plain-language HVI explanation from Task A below, labeled AI-assisted with any edits noted.

---

## Using AI in this lab

### Task A — Explain the concept

Open Claude (claude.ai) or ChatGPT and paste:

> "In one paragraph written for a public health professional without a GIS background, explain what a heat vulnerability index is, how it is typically constructed, and two methodological limitations a public health practitioner should keep in mind when interpreting it."

Read critically. Edit for accuracy. Label the output **AI-assisted** and note your edits.

### Task B — Compare classification schemes

Ask the AI:

> "I am making a choropleth map of a heat vulnerability index for Philadelphia census tracts in QGIS. Suggest two classification schemes I might use and explain the trade-offs of each for communicating spatial health inequity."

Use the response to inform your thinking. **You** decide which scheme to use and explain your choice in the deliverable. The AI explains trade-offs; it does not make the analytic decision.

> **Guardrail — required reading before using AI in this course:**
> AI assists with three things only: **scaffolding** (drafting text, QGIS expressions, or structure), **explaining** (concepts and errors), and **critiquing** (your map or interpretation). AI does **not** select datasets, make analytic decisions, or interpret rare or small-area events. Never paste patient names, addresses, case IDs, or any other identifiable health information into a public AI tool. Label all AI-assisted output in your deliverables.

---

## ArcGIS equivalents

Students on the ArcGIS Pro track complete the same lab using the tools below. The matching Pro version (Lab 1) and ArcGIS Online version (Lab 1o) exist for those platforms.

| QGIS operation | ArcGIS Pro equivalent |
|---|---|
| **Project ▸ New** (saves as `.qgz`) | New Map project (saves as `.aprx`) |
| **Layer ▸ Data Source Manager** or Browser drag | **Map** tab ▸ **Add Data** |
| **Layers** panel | **Contents** pane |
| **Browser** panel | **Catalog** pane |
| **XYZ Tiles ▸ OpenStreetMap** / QuickMapServices | Default basemap or Living Atlas basemap |
| **Layer Properties ▸ Symbology ▸ Graduated** | **Symbology** pane ▸ **Graduated Colors** |
| **Layer Properties ▸ Symbology ▸ Categorized** | **Symbology** pane ▸ **Unique Values** |
| **View ▸ Measure ▸ Measure Line** | **Map** tab ▸ **Measure ▸ Measure Distance** |
| **Project ▸ New Print Layout** | **Insert** tab ▸ **New Layout** |
| **Layout ▸ Export as PDF** | **Share** tab ▸ **Export Layout** |
| Free; no credits or license needed | Requires organizational license |

---

## Check your understanding

1. You add a polygon layer and it covers all other layers. How do you fix this in QGIS without removing the layer?
2. You move your lab folder to a different computer and the project cannot find its data. What QGIS setting prevents this, and where do you enable it?
3. A colleague says Equal Interval is always better because it shows the true data range. Another says Quantile is always better because it balances class counts. Who is right, and under what circumstances?
4. You measure 1.4 miles straight-line from a high-HVI tract to the nearest pool. What does this tell you — and what does it **not** tell you — about residents' ability to access that pool?
5. The FEMA flood zone layer and the GSI layer overlap in some areas but not others. What might spatial overlap (or absence of it) suggest about where stormwater infrastructure investment has been directed?

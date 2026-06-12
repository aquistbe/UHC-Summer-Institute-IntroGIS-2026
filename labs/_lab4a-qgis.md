## Public health question

Federally Qualified Health Centers (FQHCs) are a primary safety-net resource for uninsured and underinsured Philadelphians. But proximity on a map does not equal access. If residents cannot reach an FQHC within a reasonable travel time, that center does not serve them in practice. This lab asks: what share of West Philadelphia residents live more than a 15-minute trip from the nearest FQHC? And which residential areas fall entirely outside reasonable service reach?

## What you will learn

- Use the native QGIS Processing **Network analysis** tools — **Service area (from layer)** and **Shortest path (point to point)** — on a road-network line layer
- Understand travel cost as distance (meters) or time (minutes) depending on whether a speed field is present
- Convert service-area output nodes into catchment polygons using **Concave hull** or **Convex hull**
- Estimate covered and uncovered tract population with **Join attributes by location (summary)** and **Select by location**
- Install and use the **ORS Tools** plugin for true drive-time isochrones as an enhanced option
- Produce a finished map in the QGIS **Print Layout** designer

## What you will produce

- A map showing 15-, 30-, and 45-minute service-area catchments around Philadelphia FQHCs, symbolized by ring with a clear legend (native path or ORS Tools)
- A **Shortest path** result connecting one sample residential point to its nearest FQHC, with distance or travel-time cost in the attribute table
- A summary table showing estimated tract population falling outside the 15-minute catchment
- A 150-word written interpretation answering the public health question
- A QGIS project file (`Lab04a_NetworkAccess.qgz`) and a **Print Layout** export (PDF or PNG)

## Data

All files below are in the instructor-shared folder for Lab 4a. Substitute the actual path your instructor provides.

| Layer name | File name | Description | Source |
|---|---|---|---|
| FQHC locations | `phl_FQHC.gpkg` | Point locations of FQHCs in Philadelphia | HRSA data.hrsa.gov — instructor-provided |
| Road network | `phl_roads_osm.gpkg` | Clipped OSM road-network line layer for Philadelphia | OpenStreetMap via instructor — instructor-provided |
| Tract population | `phl_tracts_pop.gpkg` | ACS 5-year tract boundaries with total population field `TOT_POP` | Census TIGER + ACS — instructor-provided |
| Sample resident points | `westphilly_sample_pts.gpkg` | Random sample of 50 residential address points in West Philadelphia | Instructor-provided from PASDA address points |

> **Note:** All files are provided as GeoPackages (`.gpkg`), QGIS's preferred working format. If your instructor supplies shapefiles instead, the steps are identical — just substitute the `.shp` filenames.

---

## Before you begin

**Prerequisites:** Complete Labs 1–3. You should be able to add layers, use the Processing Toolbox, reproject layers, and run Select by Location.

**QGIS version:** These steps target **QGIS 3.34 LTR**. QGIS is free and open-source — no credits, no organizational account, no licensing barriers.

**Time estimate:** 90–120 minutes for the native path; add 20 minutes if you complete the ORS Tools enhanced option.

**Plugins to install (do this before starting):**

1. Open **Plugins ▸ Manage and Install Plugins…**
2. Search for **ORS Tools** and click **Install Plugin**. This plugin is needed for Part 4 (enhanced option) only. It requires a free API key from openrouteservice.org — register now if you plan to complete Part 4.
3. Optionally install **QuickMapServices** for a background basemap.

**CRS setup:** Set your project CRS to **NAD83 / Pennsylvania South (ftUS) — EPSG:2272** via **Project ▸ Properties ▸ CRS**. The road-network layer and all analysis outputs should share this CRS for accurate distance calculations. Use **Processing ▸ Reproject Layer** (or right-click a layer ▸ **Export ▸ Save Features As…** with a target CRS) to align any layer that differs.

**Create the project:** Open QGIS. Choose **Project ▸ New**. Save immediately as `Lab04a_NetworkAccess.qgz` in your working folder. Add all four data layers using **Layer ▸ Add Layer ▸ Add Vector Layer** or by dragging from the **Browser** panel.

---

## Part 1: Explore the road-network layer

Before running any analysis, inspect the road network to understand its attributes. The native QGIS network tools use a line layer directly — no special network dataset build step is required.

1. Open the **Attribute Table** of `phl_roads_osm` (press **F6** or right-click the layer ▸ **Open Attribute Table**).
2. Look for a speed or travel-time field (common OSM field names: `maxspeed`, `speed_kph`, `highway`). If a numeric speed field exists, the native tool can calculate travel time. If not, the tool will use distance as the cost.
3. Close the table. In the **Layers** panel, note the layer CRS shown in the status bar at the bottom of the QGIS window. Confirm it matches your project CRS (EPSG:2272 or equivalent).

> **Public health note:** Network distance and travel time are not the same thing, and neither equals perceived access for safety-net patients. A 15-minute drive time assumes a personal vehicle and a clear road. Many FQHC patients rely on transit, walk, or use paratransit. Drive-time service areas are a useful first screen and a common standard in health planning, but they overestimate access for car-free households. Note this limitation in your written interpretation.

---

## Part 2: Service area (from layer) — native tool

The **Service area (from layer)** tool computes all road segments reachable from each start point within a specified cost (distance or time). The output is a set of nodes and lines representing the extent of the reachable network, not a filled polygon — you will convert it to a catchment polygon in Part 3.

### Run the service area

1. Open the **Processing Toolbox** (**Processing ▸ Toolbox**, or Ctrl+Alt+T).
2. In the search box, type `service area`. Under **Network analysis**, double-click **Service area (from layer)**.
3. Fill in the dialog with the following parameters:

| Parameter | Value |
|---|---|
| Vector layer representing network | `phl_roads_osm` |
| Vector layer with start points | `phl_FQHC` |
| Path type to calculate | Shortest (or Fastest if a speed field exists) |
| Travel cost (distance or time) | `1500` (meters, roughly 15 min walking) — see note below |
| Output geometry type | Both nodes and lines |

> **Note:** The native tool's **Travel cost** is in the layer's distance units (meters for EPSG:26918; feet for EPSG:2272) when using **Shortest** path type, or in hours when using **Fastest** with a speed field. If your road layer is in feet (EPSG:2272) and you want a ~15-minute walk, use approximately `6000` feet. If your layer has a speed field and you select **Fastest**, enter `0.25` for 15 minutes (0.25 hours). Confirm the units with your instructor before running.

4. Leave **Output** (nodes) and **Output lines** as temporary layers for now. Click **Run**.

*[Figure: Service area dialog filled with phl_roads_osm and phl_FQHC, travel cost entered]*

5. Two temporary layers appear: **Service area (lines)** and **Service area (nodes)**. The lines layer shows all road segments reachable from the FQHC points. Turn off the road network layer to see the result more clearly.

*[Figure: Service area lines radiating outward from FQHC locations across the Philadelphia street grid]*

### Run for multiple cutoffs

The native tool runs one cost at a time. To produce 15-, 30-, and 45-minute rings, run the tool three times with three travel-cost values. Name each output descriptively (e.g., `service_nodes_15`, `service_nodes_30`, `service_nodes_45`).

6. Re-open **Service area (from layer)**. Change the travel cost to the 30-minute equivalent and click **Run**. Repeat for 45 minutes. Save each output to your working folder: right-click the temporary layer ▸ **Export ▸ Save Features As…** ▸ format GeoPackage, file name `service_nodes_30.gpkg`.

---

## Part 3: Build catchment polygons and estimate coverage

The service-area tool outputs points (nodes) along the reachable network edge, not a filled polygon. You will convert these into a catchment polygon using **Concave hull**, then subtract the inner ring to produce non-overlapping bands.

### Create catchment polygons

1. In the **Processing Toolbox**, search for `concave hull`. Open **Concave hull (alpha shapes)** under **Vector geometry**.
2. Set **Input layer** to `service_nodes_15`. Set **Alpha** to `0.3` (lower values produce tighter hulls; adjust if the polygon has holes). Click **Run**.

*[Figure: Concave hull polygon covering the reachable road network from FQHC points]*

3. Repeat for `service_nodes_30` and `service_nodes_45`.
4. To produce non-overlapping rings (like the ArcGIS Pro "Rings" option), use **Vector overlay ▸ Difference**: subtract the 15-minute hull from the 30-minute hull to get the 15–30 ring; subtract the 30-minute hull from the 45-minute hull to get the 30–45 ring.

> **Note:** **Convex hull** (also in the Processing Toolbox) is faster but produces a simpler, more inflated shape. Use **Concave hull** for a more realistic catchment boundary. If the concave hull produces unexpected holes, increase the Alpha value slightly.

### Symbolize the rings

5. Open **Properties ▸ Symbology** for each ring layer. Assign distinct fill colors with ~50% transparency: green for 0–15, yellow for 15–30, orange for 30–45. Set the border color to match the fill.
6. Order the layers in the **Layers** panel so the 0–15 ring is on top, followed by 15–30, then 30–45, with tract boundaries and the basemap below.

### Estimate uncovered population

7. In the **Processing Toolbox**, search for `select by location`. Open **Select by location** under **Vector selection**.

| Parameter | Value |
|---|---|
| Select features from | `phl_tracts_pop` |
| Where the features | have their centroid in |
| By comparing to features from | `service_hull_15` (the 15-minute catchment polygon) |
| Modify current selection by | Creating new selection |

8. Click **Run**. The tracts whose centroids fall inside the 15-minute catchment are selected.
9. Open the **Attribute Table** of `phl_tracts_pop` (F6). Click **Show Selected Features** at the bottom-left of the table. Note the values in the `TOT_POP` field. To sum them: in the **Field Calculator** (the abacus icon), use the expression `sum("TOT_POP")` in the preview box — or simply read the sum from the bottom status bar of the attribute table, which shows **Sum** for numeric fields when rows are selected.
10. Record the covered population. Then on the **Attribute Table** toolbar, click **Invert Selection**. Record the uncovered population the same way.
11. Calculate: `uncovered / (covered + uncovered) × 100`. Record this figure for your deliverables.

> **Warning:** Using tract centroids to assign coverage oversimplifies. A large tract may have its centroid inside the catchment while a portion of its residents live outside it. Block-level population or areal interpolation would produce a more precise estimate — methods beyond this lab's scope but worth noting in your interpretation.

> **Public health note:** This population estimate is a rough upper bound on coverage. It assumes all residents drive or walk, that the road network is accessible at all times, and that travel time is the only barrier. In practice, insurance status, clinic hours, language access, and patient load all constrain effective access. A travel-time service area is a first screen, not a complete access assessment.

---

## Part 4: Shortest path (point to point) — native tool

The **Shortest path (point to point)** tool finds the least-cost route between one start and one end point. Use it to connect a single sample residential point to its nearest FQHC.

1. In the **Processing Toolbox**, search for `shortest path`. Open **Shortest path (point to point)** under **Network analysis**.
2. Fill in the dialog:

| Parameter | Value |
|---|---|
| Vector layer representing network | `phl_roads_osm` |
| Path type to calculate | Shortest (or Fastest if speed field exists) |
| Start point (x, y) | Click the `…` button and pick one residential point from `westphilly_sample_pts` on the map |
| End point (x, y) | Click the `…` button and pick the nearest FQHC point on the map |

3. Under **Advanced parameters**, if a speed field exists in `phl_roads_osm`, set **Speed field** to that field name. Otherwise set **Default speed (km/h)** to `40` (a rough urban driving average) to get a time-cost output.
4. Click **Run**. A new line layer appears showing the route.
5. Open the attribute table of the output layer. It contains the start coordinates, end coordinates, and a **cost** field. If you used **Fastest** path type, the cost is in hours (multiply by 60 for minutes). If you used **Shortest**, the cost is in distance units.
6. In the **Field Calculator**, add a new field `cost_min` (decimal) with the expression `"cost" * 60` to convert hours to minutes if needed.

*[Figure: Single shortest-path route from a West Philadelphia residential point to its nearest FQHC]*

> **Note:** The native **Shortest path** tool handles one origin-destination pair at a time. To find the nearest FQHC for many residential points simultaneously, use the enhanced ORS Tools approach in Part 5, or iterate through points using the QGIS **Batch** processing option (the gear icon in the tool dialog).

---

## Part 5 (Enhanced): ORS Tools drive-time isochrones

The **ORS Tools** plugin connects QGIS to the OpenRouteService API and produces true drive-time isochrones — polygons that account for road speed, turn restrictions, and one-way streets. Results are more realistic than distance-buffered catchments from the native tool.

> **Note:** ORS Tools sends the FQHC coordinate points to the OpenRouteService servers to compute isochrones. This is appropriate for facility locations (public data). Do not use this workflow with patient addresses, home locations, or any PHI. The free ORS tier allows sufficient requests for a class exercise (up to 500 isochrone requests per day).

### Set up the plugin

1. In the QGIS menu, go to **Web ▸ ORS Tools ▸ Provider Settings**.
2. Paste your free ORS API key (obtained from openrouteservice.org) into the **API Key** field. Click **OK**.

*[Figure: ORS Tools provider settings dialog with API key field]*

### Run isochrones

3. Go to **Web ▸ ORS Tools ▸ Isochrones ▸ Isochrones From Layer**.
4. Fill in the dialog:

| Parameter | Value |
|---|---|
| Input Point Layer | `phl_FQHC` |
| Travel Mode | `driving-car` |
| Dimension | `time` |
| Isochrone Ranges (comma-separated, seconds) | `900,1800,2700` (= 15, 30, 45 min) |

5. Click **Run**. A polygon layer appears with one feature per FQHC per time band, with attributes including the range value and the facility ID.

*[Figure: ORS Tools isochrones — three concentric drive-time rings around each FQHC in Philadelphia]*

6. To produce city-wide coverage polygons rather than individual per-facility polygons, use **Processing ▸ Toolbox ▸ Dissolve**. Set **Dissolve field** to the range/value attribute to merge all 15-minute polygons into one shape, all 30-minute polygons into another, and so on.
7. Repeat the population coverage estimate from Part 3 (steps 7–11) using the ORS 15-minute dissolved polygon instead of the native hull.

---

## Part 6: Export a Print Layout map

1. Go to **Project ▸ New Print Layout**. Name it `Lab04a_Map`.
2. In the layout, click **Add Item ▸ Add Map**. Draw a map frame over most of the page.
3. In the **Item Properties** panel (right side), click **Set Map Extent to Map Canvas** to match the current view.
4. Add supporting items: **Add Item ▸ Add Legend**, **Add Item ▸ Add Scale Bar**, **Add Item ▸ Add North Arrow**, **Add Item ▸ Add Label** (for title, your name, and date).
5. In the Legend item properties, uncheck **Auto update** and remove layers not needed in the legend (road network, raw service nodes). Keep only the service-area rings, FQHC points, and tract boundaries.
6. Export: **Layout ▸ Export as PDF** (or **Export as Image** for PNG). Save to your working folder as `Lab04a_Map.pdf`.

*[Figure: Completed Print Layout showing service-area rings, FQHC points, tract boundaries, legend, scale bar, and north arrow]*

---

## Deliverables

Submit the following to your instructor:

1. **QGIS project file:** `Lab04a_NetworkAccess.qgz` with all layers saved (not temporary).
2. **Layout export (PDF or PNG):** A single-page map showing the three service-area catchment rings (15, 30, 45 min) around Philadelphia FQHCs, with tract boundaries and at least one shortest-path route. Include a title, legend, north arrow, and scale bar. Label the map with your name and the date.
3. **Summary table:** A small table (typed in your write-up) with: total Philadelphia tract population, population in tracts covered within 15 minutes, population in tracts not covered, and percent uncovered.
4. **150-word written interpretation:** Answer the public health question in plain language. Address: (a) what share of West Philadelphia residents appear underserved by travel-time access, (b) at least one limitation of the travel-time method for a safety-net population, and (c) one policy or programmatic implication of your finding.
5. **Label any AI-assisted output** with the prompt you used and a note on how you verified the result (see Using AI in this lab).

---

## Using AI in this lab

**Suggested use — translating the access question into a workflow:** Before starting, you may ask an LLM: *"I want to estimate the share of residents more than 15 minutes from an FQHC using QGIS 3.34 native network analysis tools. Give me a step-by-step tool sequence."* Use the response to orient yourself, then follow this handout's steps — which have been verified against QGIS 3.34 documentation — rather than the AI output directly.

**Suggested use — writing QGIS expressions:** If you need to calculate travel time in minutes from a cost field in hours, you may ask: *"Write a QGIS Field Calculator expression to convert a field called `cost` from hours to minutes."* Paste the expression into the Field Calculator, verify it produces the expected values in a few rows, then run it.

**Suggested use — troubleshooting:** If your concave hull polygon has unexpected holes or your service area looks wrong (e.g., extends across the river), ask: *"In QGIS 3.34, why might my service area (from layer) result extend into areas with no road segments?"* Use the explanation to guide your troubleshooting.

**Guardrails — required:**

- AI helps with three things only: scaffolding (draft steps, expressions, text), explaining (concepts, errors), and critiquing (your map, your interpretation).
- AI does not choose your datasets, make analytic decisions, or interpret results for your specific study area.
- Never paste patient addresses, identifiable health records, or any PHI into a public LLM. ORS Tools coordinates should only be facility locations, never patient locations.
- Any map, table, or text produced with AI assistance must be labeled as such in your submission, along with the prompt used and how you verified the output.

---

## ArcGIS equivalents

The QGIS Pro version of this lab (`Lab04a_NetworkAnalysis_PH.md`) and an ArcGIS Online version also exist for those platforms.

| QGIS (this lab) | ArcGIS Pro equivalent | ArcGIS Online equivalent |
|---|---|---|
| Processing ▸ **Service area (from layer)** | Network Analyst ▸ **Service Area** solve layer | Network Analysis ▸ **Generate Service Areas** tool |
| Processing ▸ **Shortest path (point to point)** | Network Analyst ▸ **Route** solve layer | Network Analysis ▸ **Find Routes** tool |
| **ORS Tools** isochrones (drive-time) | Network Analyst Service Area, Driving Time mode | Ready-to-Use **Generate Travel Areas** (uses credits) |
| **Concave/Convex hull** on service nodes | Built into Service Area output as polygons | Built into output |
| **Select by location** + attribute table sum | **Select By Location** + **Summary Statistics** | Query widget + statistics |
| **Join attributes by location (summary)** | **Spatial Join** tool | **Join Features** tool |
| **Print Layout** PDF export | ArcGIS Pro **Layout** view PDF export | ArcGIS Online **Print** widget or Map Viewer export |
| Free, no account needed | Requires ArcGIS Pro license + Network Analyst extension | Requires ArcGIS Online account; credits consumed for routing |

---

## Check your understanding

1. The native QGIS **Service area (from layer)** tool outputs nodes and lines, not filled polygons. Why is this, and what processing step converts those outputs into a usable catchment polygon?
2. A colleague says: "I just buffered the FQHCs by half a mile — that's basically the same as a 15-minute catchment." Under what conditions would a straight-line buffer and a network service area give similar results? When would they diverge most in West Philadelphia?
3. The **Travel cost** parameter in the native tool is in different units depending on the **Path type** selected. Explain what units you would enter for a 15-minute catchment if using **Shortest** (distance) path type on a layer in EPSG:2272 (feet), versus **Fastest** (time) path type with a speed field.
4. ORS Tools sends coordinates to external servers. Explain why this is acceptable for FQHC locations but would be inappropriate if you substituted patient home addresses.
5. Your population estimate used tract centroids to assign coverage. Name one alternative approach that would produce a more accurate estimate of the uncovered population, and describe the trade-off it involves.

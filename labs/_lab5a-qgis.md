## Public health question

Philadelphia has only a handful of EPA AQS PM2.5 monitors, and they are not evenly distributed across the city. From these sparse point measurements, can we estimate what summer PM2.5 concentrations look like everywhere — and, just as importantly, where should we trust that estimate?

This lab uses spatial interpolation to generate a continuous PM2.5 surface from monitor points. You will run IDW at multiple power settings, TIN interpolation, and Ordinary Kriging (via the SAGA provider), clip surfaces to the city boundary, and map where predictions are least reliable.

---

## What you will learn

- Load point and polygon layers and inspect the attribute table for the value field
- Run **IDW Interpolation** from the Processing Toolbox, varying the distance coefficient P (1, 2, 3)
- Run **TIN Interpolation** as a deterministic contrast to IDW
- Run **SAGA ▸ Ordinary Kriging** and understand what a variogram model represents
- Clip interpolated rasters to the city boundary using **Clip Raster by Mask Layer**
- Apply consistent symbology across surfaces for honest visual comparison
- Generate a proximity raster to map where predictions are least reliable

---

## What you will produce

1. Three IDW rasters clipped to Philadelphia (P = 1, 2, 3), a TIN surface, and an Ordinary Kriging surface — all in one Print Layout
2. A distance-to-monitor raster (uncertainty proxy), clipped to Philadelphia, as a sixth panel
3. Exported Print Layout PDF (`Lab05a_Comparison.pdf`) with legend, title, and your name
4. A 150-word written interpretation: which surface you would use for a health analysis, why, and where predictions are least reliable

---

## Data

All data are in the course data folder your instructor has shared. Confirm the path at the start of lab.

| Layer | Description | Source |
|---|---|---|
| `phl_air_monitors.gpkg` | EPA AQS PM2.5 monitor points; field `PM25_MEAN` = summer mean concentration (µg/m³) | Instructor-provided (EPA AirNow/AQS) |
| `phl_boundary.gpkg` | Philadelphia city boundary polygon for masking output rasters | Instructor-provided (TIGER/Line via PASDA) |

> **Note:** Both layers are projected to **NAD83 PA State Plane South (EPSG:2272, US feet)**. Set your project CRS to match: **Project ▸ Properties ▸ CRS**, search for `2272`, and click **OK**. Interpolation tools compute distances — a projected CRS is required for meaningful results.

---

## Before you begin

**Software:** QGIS 3.34 LTR — free and open-source, no license credits required. Download from qgis.org if needed.

**No plugins required.** IDW and TIN are native Processing tools; Ordinary Kriging runs through the SAGA provider bundled with QGIS. If SAGA tools are missing, go to **Processing ▸ Options ▸ Providers** and enable SAGA.

**Prerequisites:** Adding layers, Processing Toolbox, basic symbology (Labs 1–3).

**Estimated time:** 90–120 minutes.

Open the **Processing Toolbox** now: **Processing ▸ Toolbox** (Ctrl+Alt+T).

---

## Part 1: Project Setup and Data Inspection

1. Open QGIS 3.34. Go to **Project ▸ New**.

2. Set the project CRS: click the CRS indicator (bottom-right of the window), search for `2272`, select **NAD83 / Pennsylvania South**, click **OK**.

3. Add data via **Layer ▸ Add Layer ▸ Add Vector Layer** (or drag from the **Browser** panel): add `phl_air_monitors.gpkg` and `phl_boundary.gpkg`.

4. Right-click `phl_air_monitors` ▸ **Open Attribute Table** (F6). Locate the field `PM25_MEAN` — summer mean PM2.5 in µg/m³. Note the value range and close the table.

   > **Public health note:** The EPA annual PM2.5 NAAQS is 9 µg/m³ (revised 2024); the 24-hour standard is 35 µg/m³. Elevated concentrations disproportionately affect children, older adults, and people with pre-existing lung or heart disease.

5. Symbolize `phl_air_monitors` by size. Right-click the layer ▸ **Properties ▸ Symbology**. Change the render type to **Graduated**, set **Value** to `PM25_MEAN`, and choose a size-based method so higher values appear as larger circles. Click **Apply** and **OK**.

   *[Figure: Monitor points symbolized by PM25_MEAN as graduated circles, with phl_boundary outline visible]*

   > **Warning:** You have very few monitor points — deliberately. Interpolation will produce a smooth surface regardless of sample size, but smoothness is not accuracy. A smooth surface spanning a data gap is a model prediction, not a measurement.

6. Symbolize `phl_boundary` as an outline only: right-click ▸ **Properties ▸ Symbology ▸ Simple Fill** ▸ transparent fill, black stroke. Click **OK**.

7. Save: **Project ▸ Save As…**, name it `Lab05a_Interpolation.qgz`.

---

## Part 2: IDW Interpolation — Three Power Settings

IDW estimates a value at any location as a distance-weighted average of nearby points. Weight decreases with distance raised to the power P; higher P values give more influence to the nearest monitors.

### Run IDW — Power 1

8. In the Processing Toolbox, search for **IDW Interpolation** (**Interpolation ▸ IDW Interpolation**). Set: **Vector layer** = `phl_air_monitors`; **Interpolation attribute** = `PM25_MEAN`; **Distance coefficient P** = `1.0`; **Extent** = Calculate from Layer ▸ `phl_boundary`; **Pixel size X/Y** = `100`; output file `IDW_P1.tif`. Click **Run**.

   *[Figure: IDW P=1 raster covering Philadelphia, with monitor points overlaid]*

9. Rename the output layer `IDW_Power1` (double-click the name in the **Layers** panel).

### Run IDW — Power 2 and Power 3

10. Repeat step 8 twice: once with P = 2.0 (output `IDW_P2.tif`, layer `IDW_Power2`) and once with P = 3.0 (output `IDW_P3.tif`, layer `IDW_Power3`).

    > **Note:** Higher P values give sharply more weight to the nearest monitor. With sparse data, high P values produce "bull's-eye" artifacts around each station that have no physical meaning. Comparing P = 1, 2, and 3 makes this effect visible.

### Clip IDW Surfaces to the City Boundary

11. Search for **Clip Raster by Mask Layer** (**GDAL ▸ Raster extraction**). Run it three times — one per IDW raster — with **Mask layer** = `phl_boundary`, Nodata = `-9999`, outputs `IDW_P1_clip.tif`, `IDW_P2_clip.tif`, `IDW_P3_clip.tif`. Uncheck the unclipped layers when done.

---

## Part 3: TIN Interpolation

TIN connects sample points into triangles and interpolates linearly within each. Values at monitors are honored exactly; areas between monitors get flat-faced patches — a different visual character from IDW.

12. In the Processing Toolbox, search for **TIN Interpolation** (**Interpolation ▸ TIN Interpolation**). Set: **Vector layer** = `phl_air_monitors`, **Interpolation attribute** = `PM25_MEAN`, **Method** = Linear, **Extent** from `phl_boundary`, **Pixel size** = `100`, output `TIN_interp.tif`. Click **Run**. Rename the output `TIN_Interp`.

13. Clip to `phl_boundary` using **Clip Raster by Mask Layer** (same settings as Part 2). Save as `TIN_clip.tif`.

   > **Note:** TIN makes no assumptions about spatial autocorrelation and provides no uncertainty estimate. Compare it visually to IDW — are both telling a consistent story about where PM2.5 is highest?

---

## Part 4: Ordinary Kriging via SAGA

Kriging weights observations using the spatial autocorrelation structure of the data, modeled through a variogram. Unlike IDW, it produces both a predicted value and a prediction variance at each location. Native QGIS does not include kriging; this step uses the SAGA provider (enable under **Processing ▸ Options ▸ Providers** if needed).

14. In the Processing Toolbox, expand **SAGA ▸ Spatial and Geostatistics** (or search for **Ordinary Kriging**). Double-click **Ordinary Kriging**.

15. Set: **Points** = `phl_air_monitors`; **Attribute** = `PM25_MEAN`; **Variogram model** = Spherical; **Cell size** = `100`; extent matching `phl_boundary`. Name the prediction output `Kriging_pred.tif` and retain the variance output `Kriging_var.tif` — this is the uncertainty surface. Click **Run**.

   > **Public health note:** The variogram **range** (where the curve levels off) is the distance beyond which monitors no longer predict each other well. If the range is small relative to gaps between monitors, predictions in those gaps carry high uncertainty — typical of sparse air-quality networks.

16. Clip the prediction raster to `phl_boundary`. Save as `Kriging_clip.tif`. Also clip the variance output; save as `Kriging_variance_clip.tif`. You will display this in Part 5.

   > **Note — QGIS limitation:** ArcGIS Geostatistical Analyst provides automated cross-validation (RMSE, standardized RMS) inside its Kriging Wizard. QGIS/SAGA Ordinary Kriging does not. For formal cross-validation, use R (`gstat` + `sf`). For this lab, compare surfaces visually and discuss the trade-offs conceptually.

---

## Part 5: Symbolize and Compare the Surfaces

### Apply consistent symbology

17. Right-click `IDW_P1_clip` ▸ **Properties ▸ Symbology**. Set render type to **Singleband pseudocolor**, color ramp **YlOrRd**, **Mode** = Natural Breaks (Jenks), 5 classes. Note the Min and Max values.

18. Apply the same ramp and Min/Max to `IDW_P2_clip`, `IDW_P3_clip`, `TIN_clip`, and `Kriging_clip`. Identical color-to-concentration mapping is required for honest comparison.

    *[Figure: Symbology panel for IDW_P1_clip showing YlOrRd ramp, min/max values, and 5 classes]*

### Build a distance-to-monitor uncertainty surface

19. Search for **Rasterize (Vector to Raster)** (**GDAL ▸ Vector conversion**). Set input to `phl_air_monitors`, burn value = 1, extent and resolution matching `IDW_P1_clip`, output `monitors_raster.tif`. Then search for **Proximity (Raster Distance)** (**GDAL ▸ Raster analysis**), run on `monitors_raster.tif`, output `dist_to_monitor.tif`.

20. Clip `dist_to_monitor.tif` to `phl_boundary`. Save as `dist_to_monitor_clip.tif`. Symbolize with a sequential ramp — light near monitors, dark far away.

    > **Warning:** A smooth surface gives no visual cue about reliability. A neighborhood that looks PM2.5-low on your map may have no monitor within several kilometers. The distance-to-monitor raster is the essential companion layer.

    > **Public health note:** Modeled exposure is not measured exposure. Do not use interpolated surfaces for regulatory decisions, health alerts, or individual-level exposure characterization. Appropriate uses: exploratory hypothesis generation, identifying monitor gaps, contextualizing community health data — with explicit uncertainty acknowledgment.

---

## Part 6: Create the Print Layout

21. Go to **Project ▸ New Print Layout**, name it `Lab05a_Comparison`, click **OK**.

22. Right-click the canvas ▸ **Page Properties** ▸ **Landscape**, Letter size.

23. Use **Add Item ▸ Add Map** to draw six frames in a 2 × 3 grid: IDW P=1, P=2, P=3 (top); TIN, Kriging, Distance to Monitor (bottom). Set the **Map** source for each frame in **Item Properties** and lock it.

24. Add a shared legend (**Add Item ▸ Add Legend**): uncheck **Auto update**, keep only the PM2.5 ramp, label it "PM2.5 (µg/m³)." Add a title, your name, date, north arrow, and scale bar.

    *[Figure: Completed Print Layout with six map panels, legend, title, and scale bar]*

25. Export: **Layout ▸ Export as PDF…**, filename `Lab05a_Comparison.pdf`.

---

## Deliverables

1. **Layout PDF** (`Lab05a_Comparison.pdf`) — six panels (IDW P=1, P=2, P=3; TIN; Kriging; distance to monitor), legend, title, your name, and date.
2. **Method notes** — brief bulleted observations on visual differences across IDW P values and between IDW, TIN, and kriging. No formal statistics required.
3. **150-word interpretation** — which IDW power was most plausible with sparse data and why; how TIN differed from IDW; where in Philadelphia predictions are least reliable and why that matters for health equity. Include your name.

---

## Using AI in this lab

**Bounded AI task — explain the trade-off:** Ask an AI assistant (e.g., Claude):

*"What is the main practical difference between Inverse Distance Weighting and kriging for estimating air pollution concentrations, and what are the trade-offs for a public health analyst working with sparse monitor data?"*

Compare the response to Bolstad & Manson, Ch. 12. Note one point the AI explained clearly and one point where the textbook added nuance the AI missed.

**Guardrails:**

- AI is useful for **explaining** concepts (variogram range, RMSE), **scaffolding** a plain-language summary, and **critiquing** your written interpretation for clarity.
- AI does **not** choose the appropriate interpolation method, assess surface accuracy, or interpret spatial health patterns.
- Never paste identifiable data into a public AI tool.
- Label all AI-assisted output: "Drafted with AI assistance and revised by the student."

---

## ArcGIS equivalents

| QGIS operation | ArcGIS Pro equivalent |
|---|---|
| Processing Toolbox ▸ **IDW Interpolation** | Geostatistical Wizard ▸ Inverse Distance Weighting |
| Processing Toolbox ▸ **TIN Interpolation** | 3D Analyst Tools ▸ TIN ▸ Create TIN (then TIN to Raster) |
| SAGA ▸ **Ordinary Kriging** | Geostatistical Wizard ▸ Kriging/Cokriging ▸ Ordinary Kriging |
| Kriging cross-validation: not built in — use R `gstat` | Geostatistical Wizard cross-validation window (RMSE, standardized RMS) |
| GDAL ▸ **Clip Raster by Mask Layer** | Spatial Analyst ▸ Extraction ▸ Extract by Mask |
| GDAL ▸ **Proximity (Raster Distance)** | Spatial Analyst ▸ Distance ▸ Euclidean Distance |
| **Print Layout** (Project ▸ New Print Layout) | Insert ▸ New Layout |

Matching ArcGIS Pro and Online versions exist. The principal QGIS limitation is absent built-in kriging cross-validation; use R (`gstat`) for that. QGIS is free.

---

## Check your understanding

1. You run IDW with P = 1 and P = 3 on the same monitor data. Which surface will show sharper "bull's-eye" patterns around individual monitors, and why?

2. TIN interpolation honors the exact measured value at each monitor location, while IDW produces a weighted average. Under what circumstances might TIN be misleading in the spaces between monitors?

3. A colleague says: "This map shows PM2.5 is uniformly low in Northeast Philadelphia." What is the flaw, and how would you use the distance-to-monitor raster to respond?

4. A community organization with no nearby monitor asks you to report PM2.5 at their block. Your only data are 8 city-wide monitors. What cautions would you include in your response?

5. QGIS/SAGA Ordinary Kriging does not provide cross-validation statistics. What would you do to obtain a formal RMSE for the kriging surface, and why does that matter for using modeled exposure in an epidemiologic study?

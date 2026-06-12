## Public health question

Summer heat is the leading weather-related cause of death in the United States, but the burden falls unevenly across neighborhoods. Which Philadelphia census tracts have the highest land-surface temperatures during summer, and do those tracts also carry the highest social vulnerability? Identifying that overlap is an early step in targeting heat-health interventions — cooling centers, tree-planting programs, and outreach to isolated elderly residents.

## What you will learn

- Add and inspect a single-band GeoTIFF raster in QGIS
- Symbolize a raster with **Singleband pseudocolor** and a sequential heat ramp
- Use **Raster ▸ Raster Calculator** to convert units or build a simple index
- Reclassify a continuous raster into discrete categories with **Reclassify by table** (Processing Toolbox)
- Run **Zonal statistics** (Processing) to compute mean LST per census tract — QGIS writes results directly to the polygon layer, with no separate join step needed
- Symbolize tract polygons by a numeric field using **Graduated** colors
- Compare LST and SVI layers to identify doubly-burdened tracts
- Export a finished map from the **Print Layout** designer

## What you will produce

1. A choropleth map of Philadelphia census tracts symbolized by zonal mean LST, exported as a PDF from Print Layout
2. A side-by-side or inset comparison with SVI quartile symbology
3. *(Bonus)* A bivariate display combining LST class and SVI quartile
4. A short written interpretation (~100 words) on the spatial pattern and the limits of the analysis
5. The saved QGIS project file (`.qgz`)

## Data

All files are instructor-provided. Save them to your working folder for this lab.

| Layer name | File | Description | Source |
|---|---|---|---|
| Summer LST raster | `phl_LST_summer.tif` | Single-band Landsat 8/9 summer land-surface temperature, Philadelphia extent, values in °C | Instructor-provided (USGS Landsat / NOAA HEAT.gov) |
| Tracts with SVI | `phl_tracts_SVI.gpkg` | 2020 TIGER census tract boundaries with CDC/ATSDR SVI 2022 overall percentile field (`RPL_THEMES`) pre-joined | Instructor-provided (CDC/ATSDR SVI 2022) |

> **Note:** Both layers are in **NAD83 PA State Plane South (EPSG:2272, US feet)**. Set your project CRS: **Project ▸ Properties ▸ CRS**, search `2272`, click **OK**.

## Before you begin

**QGIS version:** This lab targets **QGIS 3.34 LTR**. QGIS is free and open-source — no license, no extension, and no credits are required. All tools are built into the default installation. This contrasts with the ArcGIS Pro version of this lab, which requires the Spatial Analyst extension, and the ArcGIS Online version, which consumes raster-analysis credits.

**Prerequisites:** Comfortable adding data, using the **Layers** panel, and opening tools from the **Processing Toolbox**. No prior raster experience required.

**Plugins needed:** None.

**Time estimate:** 90–110 minutes.

---

## Part 1: Add and Inspect the LST Raster

1. **Layer ▸ Add Layer ▸ Add Raster Layer**. Browse to `phl_LST_summer.tif` and click **Add**.
2. **Layer ▸ Add Layer ▸ Add Vector Layer**. Browse to `phl_tracts_SVI.gpkg` and click **Add**. In the **Layers** panel, drag `phl_tracts_SVI` above the raster.
3. Right-click `phl_LST_summer.tif` ▸ **Properties ▸ Information**. Note the pixel size and value range. Close the dialog.
4. Click the **Identify Features** tool and click a pixel on the raster to see its °C value.

### Symbolize with a heat ramp

5. Double-click `phl_LST_summer.tif` to open **Properties ▸ Symbology**. Change render type to **Singleband pseudocolor**.
6. Click the **Color ramp** dropdown ▸ **Create New Color Ramp ▸ Catalog: cpt-city**. Choose a light-to-dark sequential ramp such as **YlOrRd**. Click **OK**.
7. Set **Interpolation** to **Linear**, **Mode** to **Continuous**. Click **Classify**, then **Apply** and **OK**.

*[Figure: Symbology tab showing Singleband pseudocolor with YlOrRd ramp applied to the LST raster over Philadelphia]*

> **Public health note:** LST measured by satellite is a surface thermal emission, not air temperature and not personal heat exposure. High values reflect impervious surfaces, dark rooftops, and sparse tree cover — the urban heat island effect. LST is a useful spatial proxy for prioritization, not a measure of what residents experience.

---

## Part 2: Reclassify LST into Temperature Classes

Reclassification converts the continuous temperature raster into discrete integer categories for cleaner thematic mapping.

1. Open the **Processing Toolbox** (**Processing ▸ Toolbox** or Ctrl+Alt+T). Search `reclassify`. Open **Raster analysis ▸ Reclassify by table**.

2. Set parameters:

| Parameter | Value |
|---|---|
| Raster layer | `phl_LST_summer.tif` |
| Band number | 1 |
| Range boundaries | `min <= value < max` |
| Output raster | `lst_reclass.tif` in your working folder |

3. Click the ellipsis next to **Reclassification table**. Add five rows using break values from your raster statistics. Assign integer values 1 (coolest) through 5 (hottest). Click **OK**, then **Run**.

4. Open the output layer's **Symbology**, set render type to **Paletted/Unique values**, click **Classify**, and assign a light-to-dark sequential ramp matching the raw raster.

> **Note:** The reclassified raster stores class codes 1–5, not degrees Celsius. Keep `phl_LST_summer.tif` for the zonal statistics step — that tool needs continuous values.

---

## Part 3: (Optional) Raster Calculator — Unit Conversion

1. Open **Raster ▸ Raster Calculator**.
2. In the **Raster Bands** list, double-click `phl_LST_summer.tif@1` to insert it. Build the expression:

   `"phl_LST_summer.tif@1" * 9 / 5 + 32`

3. Set the output to `lst_fahrenheit.tif`. Click **OK**. Verify values are ~32–50 °F higher than the Celsius raster.

> **Warning:** Layer names in the Raster Calculator must match the **Raster Bands** list exactly, including the `@1` band suffix. Use the list to insert names rather than typing them manually.

---

## Part 4: Zonal Statistics

**Zonal statistics** summarizes raster pixel values within each polygon zone. Here the zones are census tracts and the raster is LST. QGIS writes mean LST directly back to the tract layer as a new field — no separate join step is required.

> **Public health note:** Zonal statistics is the most useful raster operation for public health spatial analysis. The same workflow applies to any continuous environmental raster: PM2.5 by census tract, NDVI (tree canopy) by neighborhood, impervious surface by block group. Learn this once and apply it to dozens of environmental justice questions.

1. In the **Processing Toolbox**, search `zonal statistics`. Open **Raster analysis ▸ Zonal statistics**.

2. Set parameters:

| Parameter | Value |
|---|---|
| Input layer | `phl_tracts_SVI` |
| Raster layer | `phl_LST_summer.tif` |
| Raster band | 1 |
| Output column prefix | `lst_` |
| Statistics to calculate | **Mean** (optionally also Min, Max) |
| Output layer | `phl_tracts_SVI_zonal.gpkg` in your working folder |

3. Click **Run**. Open the **Attribute Table** (F6) of the output layer. Scroll right to find `lst_mean`. Confirm one row per tract with plausible °C values.

*[Figure: Attribute Table showing the lst_mean field populated with temperature values per census tract]*

> **Note:** QGIS Zonal Statistics appends results directly to the output polygon layer. The ArcGIS Pro equivalent (**Zonal Statistics as Table**) creates a separate table requiring a subsequent join step.

### Symbolize tracts by mean LST

4. Open **Properties ▸ Symbology** for `phl_tracts_SVI_zonal`. Set render type to **Graduated**, **Value** to `lst_mean`, **Mode** to **Natural Breaks (Jenks)**, **Classes** to **5**. Choose the same yellow-to-red ramp from Part 1. Click **Classify**, then **Apply** and **OK**.

*[Figure: Philadelphia census tract map with graduated-color symbology by lst_mean; hottest tracts in dark red]*

---

## Part 5: Classify SVI and Build the Comparison Display

The SVI overall percentile field `RPL_THEMES` runs 0–1, where values near 1 indicate highest social vulnerability. Quartile classification divides tracts into four equal groups.

1. Duplicate the output layer: right-click `phl_tracts_SVI_zonal` ▸ **Duplicate Layer**. Rename the duplicate `tracts_SVI_symbology`.
2. Open **Properties ▸ Symbology** on the duplicate. Set render type to **Graduated**, **Value** to `RPL_THEMES`, **Mode** to **Quantile (Equal Count)**, **Classes** to **4**. Choose a light-to-dark purple ramp (e.g., **Purples**). The darkest shade = Q4, most vulnerable. Click **Classify**, then **Apply** and **OK**.

*[Figure: Census tract map symbolized by SVI quartile; darkest purple = Q4 (highest vulnerability)]*

3. Toggle layer visibility to compare the LST and SVI layers. Identify tracts that are in the hottest LST class **and** SVI Q4 — these are doubly-burdened tracts. Use **Identify Features** to record their `GEOID`, `lst_mean`, and `RPL_THEMES`.

> **Public health note:** Spatial co-occurrence of high LST and high SVI is not evidence of worse health outcomes. LST is a surface proxy, not air temperature, personal exposure, or illness. A full heat vulnerability assessment would add emergency department visit data for heat illness, air-conditioning access, cooling center proximity, and age structure. This map is a prioritization tool, not a conclusion.

### (Bonus) Bivariate display

4. Open the **Field Calculator** on `phl_tracts_SVI_zonal`. Create a new text field `bvclass`. Write an expression combining the LST class and SVI quartile — for example, assigning `'5Q4'` to a tract with the highest LST and highest SVI. Your instructor will supply break values. Symbolize by **Categorized** on `bvclass`, assigning a bivariate color grid with the darkest, most alarming color at `5Q4`.

---

## Part 6: Print Layout

1. **Project ▸ New Print Layout**. Name it `Lab04c_LST`.
2. **Add Item ▸ Add Map**. Draw a rectangle; set extent to match Philadelphia.
3. Add **Legend** (edit to show only relevant layers and rename class labels), **Scale Bar**, **North Arrow**, and a **Label** for the title and data sources.
4. **Layout ▸ Export as PDF**. Repeat for the SVI quartile map.

*[Figure: Print Layout designer with map frame, legend, scale bar, north arrow, and title]*

---

## Deliverables

1. **Map 1 — Zonal Mean LST by Census Tract.** Print Layout PDF: title, legend, north arrow, scale bar, data sources, your name. Graduated color by `lst_mean`, 5 Natural Breaks classes, yellow-to-red ramp.
2. **Map 2 — SVI Quartile by Census Tract.** Same extent. Graduated by `RPL_THEMES`, 4 quantile classes, light-to-dark purple.
3. **Written interpretation (~100 words).** Identify 2–3 doubly-burdened tracts by GEOID or neighborhood. Describe the pattern. State explicitly that LST is a surface proxy, not health outcome data.
4. *(Bonus)* **Map 3 — Bivariate Display.** Same layout requirements; include a bivariate legend.
5. **Saved project** `Lab04c.qgz`.

Label any AI-assisted output clearly.

---

## Using AI in this lab

### Suggested task: interpret the comparison map

After completing Maps 1 and 2, paste this prompt into Claude (claude.ai) or another AI assistant. Do not include PHI or identifiable data.

> "I have created two maps of Philadelphia census tracts: one showing summer land-surface temperature (LST) as a heat proxy, the other showing CDC/ATSDR Social Vulnerability Index (SVI) quartiles. High-LST, high-SVI tracts appear concentrated in [describe your observed pattern]. Draft a 200-word equity-framed summary suitable for a neighborhood advisory board presentation."

**Critical review task.** Read the AI draft and flag every sentence where the AI describes LST as heat *exposure*, implies a health outcome absent from this dataset, or makes a causal claim the data do not support. Revise to correct those overstatements. Include two sentences in your deliverable explaining what you changed and why.

**QGIS expression tip.** If you need help with the `bvclass` Field Calculator expression, ask the AI: "Write a QGIS Field Calculator expression that combines an LST class (1–5 based on these breaks: …) and SVI quartile (Q1–Q4 based on RPL_THEMES)." Test the result before using it.

### AI guardrails (always apply)

- AI assists with scaffolding, explaining tool mechanics, and critiquing — not with analytic decisions, dataset selection, or interpreting maps it cannot see.
- Never paste PHI or identifiable data into a public AI system.
- Label all AI-assisted output: "AI-assisted draft, revised by [your name]."

---

## ArcGIS equivalents

The same lab exists as **Lab 4c (ArcGIS Pro)** and **Lab 4c (ArcGIS Online)**. The table below maps QGIS tools to their Pro counterparts.

| QGIS operation | ArcGIS Pro equivalent |
|---|---|
| No extension needed | Enable **Spatial Analyst** extension (licensed) |
| **Properties ▸ Symbology ▸ Singleband pseudocolor** | **Symbology pane ▸ Classify**, color ramp |
| **Reclassify by table** (Processing Toolbox) | **Spatial Analyst ▸ Reclass ▸ Reclassify** |
| **Raster ▸ Raster Calculator** | **Spatial Analyst ▸ Map Algebra ▸ Raster Calculator** |
| **Zonal statistics** — appends fields to polygon layer directly | **Zonal Statistics as Table** — creates a separate table; requires **Add Join** |
| **Project ▸ New Print Layout** | **Insert ▸ New Layout** |
| Free; no credits | License required; ArcGIS Online raster ops consume credits |

---

## Check your understanding

1. QGIS **Zonal statistics** writes mean LST directly to the tract layer. The ArcGIS Pro equivalent creates a separate table requiring a join. What field would you use as the join key, and why does its uniqueness matter?

2. You classified LST using Natural Breaks (Jenks). A classmate used Equal Interval with the same five classes. Would the same tracts appear in the "hottest" class under both methods? Explain.

3. A colleague argues that tracts with the highest `lst_mean` values are the most dangerous for heat-related illness. What is wrong with this claim? Name two additional data sources needed to assess actual heat-health risk.

4. In the Raster Calculator, why must you include the `@1` band suffix when referencing `phl_LST_summer.tif`?

5. Why is zonal mean LST per tract more useful for public health decision-making than simply displaying the raw raster with a graduated color ramp?

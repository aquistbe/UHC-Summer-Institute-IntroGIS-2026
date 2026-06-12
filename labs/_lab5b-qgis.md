## Public health question

Extreme heat is the leading weather-related cause of death in the United States, and its burden falls unevenly across neighborhoods. Communities with high impervious cover, low tree canopy, elevated land-surface temperatures, and concentrated social vulnerability face compounding risks. Which Philadelphia census tracts carry the highest combined heat-health risk, and how do we communicate that finding responsibly? By combining environmental and social layers into a composite index, you can direct heat-mitigation resources — cooling centers, tree planting, outreach — toward the places that need them most.

---

## What you will learn

- Verify that multiple raster layers share a common CRS, extent, and cell size; align them when they do not
- Rasterize a vector polygon layer (CDC SVI tracts) to match the resolution of your environmental rasters
- Use **Reclassify by table** to convert each input to a 1–5 ordinal risk scale, and explain the value judgments in break choices and weighting
- Use the **Raster Calculator** to compute a weighted-sum composite with explicitly stated weights
- Run a sensitivity check with a second weighting scheme and compare spatial results
- Symbolize the composite with a colorblind-safe sequential ramp (ColorBrewer or Viridis)
- Build a complete professional map in **Print Layout** and export to PDF and PNG
- Apply ethical framing to a vulnerability map intended for public audiences

---

## What you will produce

1. A composite raster `heat_vuln_primary.tif` (primary weighted run)
2. A sensitivity-check raster `heat_vuln_sensitivity.tif` with a brief written comparison
3. A Print Layout exported as PDF (300 dpi) and PNG (150 dpi) with all required map elements
4. A 150-word written methods paragraph: inputs, reclassification breaks, and weights for both runs
5. AI-assisted map alt-text and critique, labeled and annotated
6. Saved QGIS project file `Lab05b_HeatVuln.qgz`

---

## Data

Files are in the course shared folder (`Lab05b/data/`). Your instructor will confirm the exact path.

| Layer name | Description | Source |
|---|---|---|
| `phl_LST_summer.tif` | Mean summer land-surface temperature (°C), Landsat 8/9, Philadelphia | Instructor-provided (Lab 4c); USGS / NOAA HEAT.gov |
| `phl_treecanopy.tif` | Percent tree canopy cover, 30 m cell, NLCD | Instructor-provided; MRLC / USFS |
| `phl_impervious.tif` | Percent impervious surface, 30 m cell, NLCD | Instructor-provided; MRLC |
| `phl_tracts_SVI.gpkg` | CDC/ATSDR Social Vulnerability Index 2022, Philadelphia tracts with geometry | CDC/ATSDR SVI; atsdr.cdc.gov/placeandhealth/svi |
| `phl_boundary.gpkg` | Philadelphia County boundary | U.S. Census Bureau / instructor |

> **Note:** The lab uses LST, tree canopy, and SVI as the three primary inputs. If your instructor substitutes impervious surface for tree canopy, the reclassification direction reverses (high impervious = high risk); the steps are otherwise identical.

---

## Before you begin

**QGIS version:** QGIS **3.34 LTR**. QGIS is free and open-source — no license costs apply.

**Prerequisites:** Labs 1–5a; Labs 4c (LST) and 4a–b (SVI join) should be complete.

**Plugins:** For the optional interactive web export in Part 7, install **qgis2web** via **Plugins ▸ Manage and Install Plugins…**

**Time estimate:** 2.5–3 hours.

**Setup:** Create a project folder, save a new project named `Lab05b_HeatVuln.qgz`, and set the project CRS to **EPSG:26918** via **Project ▸ Properties ▸ CRS**.

---

## Part 1: Verify inputs — CRS, extent, and cell size

Before any raster analysis, confirm all inputs share the same projection and spatial resolution. Mismatched rasters produce silent errors — cells that do not align yield incorrect composite values.

1. Load all five files: drag from the **Browser** panel, or use **Layer ▸ Add Layer**.
2. Right-click `phl_LST_summer` → **Properties ▸ Information**. Note the CRS (**EPSG:26918**), pixel size (30 × 30 m), and extent.
3. Repeat for `phl_treecanopy` and `phl_impervious`. All three rasters must match.
4. Right-click `phl_tracts_SVI` → **Properties ▸ Information**. If the CRS differs from EPSG:26918, reproject it now: in the **Processing Toolbox** (**Processing ▸ Toolbox**), search **Reproject layer**, set the target CRS to `EPSG:26918`, and save as `phl_tracts_SVI_26918.gpkg`.

*[Figure: Information tab for phl_LST_summer showing EPSG:26918 and 30 m pixel size]*

If rasters differ in extent or pixel alignment, use **Warp (Reproject)** (Processing Toolbox → GDAL) to resample each to the `phl_LST_summer` grid. Use **Nearest neighbour** resampling for categorical data and **Bilinear** for continuous inputs.

> **Note:** NAD83 UTM Zone 18N (meters) preserves distances and areas. Do not mix CRS or unit systems in raster arithmetic — the Raster Calculator will produce incorrect results if inputs are not pixel-aligned.

> **Public health note:** CRS mismatches are one of the most common silent errors in vulnerability mapping. If your LST and SVI layers are in different projections and you skip this check, the overlay will attribute risk to the wrong neighborhoods with no obvious warning.

---

## Part 2: Rasterize the SVI layer

The SVI is a vector polygon layer. The Raster Calculator requires raster inputs; convert it to a raster matching the LST grid.

1. In the Processing Toolbox, search **Rasterize (vector to raster)** (GDAL ▸ Vector conversion).
2. Set parameters:

   | Parameter | Value |
   |---|---|
   | Input layer | `phl_tracts_SVI.gpkg` |
   | Field to burn | `RPL_THEMES` (overall SVI percentile rank, 0–1) |
   | Resolution (X and Y) | `30` m |
   | Output extent | Calculate from `phl_LST_summer` |
   | Output | `phl_SVI_raster.tif` |

3. Click **Run**. Each 30 m cell receives the `RPL_THEMES` value of its containing tract.

> **Public health note:** `RPL_THEMES` ranges 0–1; values near 1 indicate greater vulnerability. Cells at tract boundaries take the value of the dominant tract — a known approximation. Block-group SVI data improve spatial precision if available.

---

## Part 3: Reclassify each input to a 1–5 risk scale

Reclassify each raster to a 1–5 ordinal scale where **5 = highest heat-health risk**.

| Input | Direction | Rationale |
|---|---|---|
| LST | Low temperature → 1; high → 5 | Higher temperatures increase heat exposure |
| Tree canopy | High canopy → 1; low canopy → 5 | Canopy cools; its absence increases risk |
| SVI (`RPL_THEMES`) | Low rank → 1; high rank → 5 | Higher percentile = greater social vulnerability |

> **Warning:** Classification breaks are a value judgment — they determine which tracts land in class 5. Record your break values; they must appear in your methods write-up.

In the Processing Toolbox, search **Reclassify by table** (Raster analysis). Repeat for each input:

**LST (`lst_rcls.tif`):** Use quantile breaks (five roughly equal-count bins) over the observed LST range. Assign new values 1–5 from coolest to hottest. Run **Raster layer statistics** first to see the min/max range.

**Tree canopy (`canopy_rcls.tif`):** Same quantile approach, but **reverse the new values** — highest canopy class = 1, lowest canopy class = 5.

**SVI (`svi_rcls.tif`):** Because `RPL_THEMES` is already a uniform percentile rank, use equal-interval breaks:

| Minimum | Maximum | New value |
|---|---|---|
| 0.00 | 0.20 | 1 |
| 0.20 | 0.40 | 2 |
| 0.40 | 0.60 | 3 |
| 0.60 | 0.80 | 4 |
| 0.80 | 1.001 | 5 |

For each run: set **Range boundaries** to `min <= value < max`, name the output raster, and click **Run**.

*[Figure: Reclassify by table dialog for LST showing five quantile break rows and new values 1–5]*

> **Public health note:** Break values are an analytic choice that determines which tracts appear as "high risk." Quantile breaks for LST and equal-interval breaks for SVI are defensible — but they are choices. Acknowledge them in your methods write-up.

---

## Part 4: Build the composite with Raster Calculator

Weights must sum to 1.0. State them explicitly — they are an analytic decision.

**Primary weights:**

| Input | Weight | Rationale |
|---|---|---|
| `lst_rcls` | 0.40 | Most direct driver of acute heat exposure |
| `canopy_rcls` | 0.30 | Structural cooling mechanism |
| `svi_rcls` | 0.30 | Social vulnerability mediates capacity to cope |

1. Open **Raster ▸ Raster Calculator…**
2. Enter the expression (double-click layer names from the **Raster Bands** list to insert them):

   ```
   ("lst_rcls@1" * 0.4) + ("canopy_rcls@1" * 0.3) + ("svi_rcls@1" * 0.3)
   ```

3. Set output to `heat_vuln_primary.tif`; CRS `EPSG:26918`; cell size `30`; extent from `phl_LST_summer`.
4. Click **OK**.

*[Figure: Raster Calculator dialog showing the weighted-sum expression with three reclassified inputs]*

> **Warning:** If the result has large unexpected NoData areas, confirm all three input rasters share the same extent and pixel alignment. A single raster with extra NoData cells will blank out all overlapping cells. Re-run Part 1 alignment if needed.

---

## Part 5: Sensitivity check with alternative weights

**Alternative weights:** LST 0.20, canopy 0.20, SVI 0.60. This scenario treats social vulnerability as the dominant driver, consistent with a social determinants of health framework.

Open the Raster Calculator and enter:

```
("lst_rcls@1" * 0.2) + ("canopy_rcls@1" * 0.2) + ("svi_rcls@1" * 0.6)
```

Output: `heat_vuln_sensitivity.tif`. Apply the same color ramp (Part 6) to both composites and compare: which neighborhoods move into or out of the highest-risk class when weight shifts toward SVI?

> **Public health note:** If both runs identify the same high-risk tracts, the index is robust to weight choice. If they diverge, justify your primary weights with evidence or present both maps and acknowledge the uncertainty. Showing only the result that confirms a preferred conclusion is analytic bias.

---

## Part 6: Classify and symbolize the composite

1. Right-click `heat_vuln_primary` → **Properties ▸ Symbology**.
2. Set **Render type** to **Singleband pseudocolor**, **Interpolation** to **Discrete**, **Mode** to **Equal interval**, **Classes** to **5**.
3. Open **Color ramp** → **Create new color ramp…** → **Catalog: ColorBrewer** → choose **YlOrRd** (sequential), or select **Viridis**. Map low values to pale yellow (low risk) and high values to dark red or purple (high risk).
4. Edit class labels: **1 — Lowest risk** through **5 — Highest risk**.

> **Note:** ColorBrewer and Viridis sequential ramps remain perceptible with common forms of color-vision deficiency. Avoid red-green diverging ramps for vulnerability maps shown to general audiences.

*[Figure: Symbology panel showing Singleband pseudocolor with YlOrRd ramp and five labeled classes]*

---

## Part 7: Build the Print Layout communication map

QGIS Print Layout is a full map-design environment and a core QGIS strength for producing publication-ready outputs.

1. **Create a layout:** **Project ▸ New Print Layout…**, name it `Heat_Vuln_Map`. Right-click the canvas → **Page Properties** → Letter, Landscape.
2. **Map frame:** **Add Item ▸ Add Map**. Draw a rectangle on most of the canvas. In **Item Properties**, set the map extent to Philadelphia. Leave margins for title, legend, and source text.
3. **Title:** **Add Item ▸ Add Label**. Text: `Heat-Health Vulnerability Index — Philadelphia, PA`. Sans-serif, 18–20 pt, bold. Subtitle line: `Composite of Land-Surface Temperature, Tree Canopy, and Social Vulnerability Index`.
4. **Legend:** **Add Item ▸ Add Legend**. In Item Properties, uncheck **Auto update**. Remove all layers except `heat_vuln_primary`; rename it `Heat-Health Risk`. Confirm class labels match Part 6.
5. **Scale bar:** **Add Item ▸ Add Scale Bar**. Units: Kilometers. Style: Single Box or Alternating Bar.
6. **North arrow:** **Add Item ▸ Add North Arrow**. Choose a simple style.
7. **Source and credits:** **Add Item ▸ Add Label** at the bottom of the layout:

   ```
   Data sources: CDC/ATSDR SVI 2022; USGS/NOAA Land-Surface Temperature; NLCD Tree Canopy (MRLC).
   Projection: NAD83 / UTM Zone 18N (EPSG:26918). Produced: [your name], Intro GIS for Public Health, [date].
   ```

*[Figure: Completed Print Layout with title, map frame, legend, scale bar, north arrow, and source text]*

> **Public health note:** An internal analyst map differs from a public-facing communication map. For community or media audiences, note that high-risk scores describe structural conditions of places — not the capabilities of residents. Consider whether naming specific neighborhoods serves the public health goal or risks deficit framing.

> **Warning:** Replace all auto-generated layer names (e.g., `heat_vuln_primary`) with plain-language text before exporting.

**Export:** **Layout ▸ Export as PDF…** at 300 dpi → `Lab05b_HeatVuln_[YourName].pdf`. Repeat with **Export as Image…** at 150 dpi, PNG format → `Lab05b_HeatVuln_[YourName].png`.

**Optional — interactive web map:** The **qgis2web** plugin (**Web ▸ qgis2web ▸ Create web map**) exports your map to a Leaflet or OpenLayers HTML file. Note that ArcGIS StoryMaps and Dashboards are Esri-only; the QGIS equivalents are a Print Layout PDF or a qgis2web export.

**Vector composite alternative:** Open the `phl_tracts_SVI` attribute table (F6), use the **Field Calculator** (abacus icon) to create a new field `heat_idx_primary`, and enter: `("lst_mean" * 0.4) + ("canopy_inv" * 0.3) + ("RPL_THEMES" * 0.3)` (using zonal-statistics fields from earlier labs). Symbolize with **Graduated** and a sequential ramp.

---

## Deliverables

| # | Item | Notes |
|---|---|---|
| 1 | `Lab05b_HeatVuln_[YourName].pdf` | Exported Print Layout, 300 dpi |
| 2 | `Lab05b_HeatVuln_[YourName].png` | Exported Print Layout, 150 dpi |
| 3 | `Lab05b_HeatVuln.qgz` | Saved QGIS project file |
| 4 | Written methods paragraph | 150 words max; inputs, breaks, and weights for both runs |
| 5 | Sensitivity comparison note | 2–3 sentences on which tracts changed class and what that implies |
| 6 | AI-assisted map alt-text | Label AI-drafted sentences and edits made |
| 7 | AI map critique response | List critique points; note which you acted on and why |

---

## Using AI in this lab

**Task A — Alt-text.** Share your exported PNG with Claude and prompt: *"Write one paragraph of accessibility alt-text for this heat-health vulnerability map of Philadelphia. Describe the geographic pattern, the color ramp, and the key finding. Aim for 75–100 words."* Edit any factual errors and label the output: `[AI-drafted, edited by [Your Name]]`.

**Task B — Map critique.** Prompt Claude: *"Critique this map for: (1) color choice and colorblind accessibility, (2) appropriateness of the classification method, and (3) ethical framing — does the title or legend risk stigmatizing neighborhoods? Suggest specific improvements."* List each critique point and state whether you acted on it and why.

**Guardrails:**
- AI helps with scaffolding, explaining, and critiquing. It does not make analytic decisions, choose datasets, select weights, or interpret patterns in the data.
- Never paste patient-level, address-level, or identifiable public health data into a public LLM.
- Label all AI-assisted output. Unlabeled AI output is an academic integrity violation.
- Verify every AI-generated factual claim, including any QGIS expressions, before relying on them.

---

## ArcGIS equivalents

The matching ArcGIS Pro version of this lab (Lab 5b Pro) and an ArcGIS Online version also exist for those platforms.

| QGIS step | ArcGIS Pro equivalent |
|---|---|
| **Rasterize (vector to raster)** — GDAL | **Polygon to Raster** — Conversion Tools |
| **Reclassify by table** | **Reclassify** — Spatial Analyst → Reclass |
| **Raster Calculator** weighted sum | **Weighted Overlay** or **Weighted Sum** — Spatial Analyst |
| **Warp (Reproject)** for alignment | **Project Raster** — Data Management |
| **Project ▸ New Print Layout** | **Insert ▸ New Layout** in ArcGIS Pro |
| **qgis2web** (Leaflet/OpenLayers export) | ArcGIS Online web map; ArcGIS StoryMaps (Esri-only) |

---

## Check your understanding

1. You reclassified tree canopy so that low canopy = 5 (high risk) and high canopy = 1 (low risk). Explain in one sentence why the reclassification direction is reversed compared to LST.
2. In the primary weighted sum, LST received weight 0.40 and SVI received 0.30. Name one alternative weighting scheme you could justify and state the evidence or reasoning that would support it.
3. A colleague says the map shows that residents of North Philadelphia are at high risk. Why is that framing potentially problematic, and how would you reword it?
4. What does the sensitivity analysis tell you when the primary and alternative weight runs produce nearly identical spatial patterns? What does it tell you when they diverge substantially?
5. You want to add a fifth input — distance to cooling centers — to the composite. Briefly describe the steps: what data you need, how you would derive a distance raster in QGIS, and in which direction you would reclassify it (high proximity = lower or higher risk score).

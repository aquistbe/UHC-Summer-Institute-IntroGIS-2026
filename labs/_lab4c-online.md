## Public health question

Summer heat is the leading weather-related cause of death in the United States, but the burden falls unevenly across neighborhoods. Which Philadelphia census tracts have the highest summer land-surface temperatures, and do those tracts also carry the highest social vulnerability? Identifying that overlap is an early step in targeting heat-health interventions — cooling centers, tree-planting programs, and outreach to isolated elderly residents.

## What you will learn

- Add Living Atlas layers (heat/LST imagery and SVI tracts) to a web map in ArcGIS Online Map Viewer
- Use **Summarize Within** or **Join Features** to attach a mean heat value to each census tract polygon
- Classify an attribute field using **Counts and Amounts (Color)** symbology with quantile or natural-breaks classification
- Interpret two thematic layers side by side to identify doubly-burdened tracts
- Understand why full raster zonal statistics requires additional privileges in ArcGIS Online and what the feature-based alternative achieves
- Share a finished web map and export a map image as a deliverable

## What you will produce

1. A saved, shared web map showing Philadelphia census tracts symbolized by mean summer heat value per tract (Path A) or zonal mean LST (Path B)
2. The same tract layer with an SVI quartile overlay or second map for comparison, identifying doubly-burdened tracts
3. *(Optional bonus)* A bivariate display combining heat class and SVI quartile using a two-field color scheme or an ArcGIS Instant App
4. A short written interpretation (~200 words, see the AI task below) identifying the highest heat-vulnerability tracts and the limits of this analysis

## Data

| Layer | Description | Source |
|---|---|---|
| CDC/ATSDR Social Vulnerability Index — Census Tracts | 2020 census tract boundaries with SVI 2022 overall percentile field (`RPL_THEMES`) for Pennsylvania | Living Atlas (search: "Social Vulnerability Index CDC") |
| Heat Severity or LST feature/imagery layer | A heat-related feature layer or summarized imagery layer for Philadelphia; see Path A note below | Living Atlas (search: "heat severity" or "land surface temperature") OR instructor-shared ArcGIS Online item |
| *(Path B only)* Landsat LST imagery service | Continuous LST imagery tile layer; used as input to raster zonal statistics | Living Atlas Landsat imagery service OR instructor-provided imagery layer URL |

> **Note:** The Living Atlas catalog changes over time. If your instructor has shared specific item IDs for this lab, use those instead of searching. For the heat layer, suitable Living Atlas options include the **Heat Severity Index** layer from NOAA, CDC PLACES layers, or instructor-processed Landsat summer LST shared as an ArcGIS Online hosted imagery layer. Your instructor will confirm which layer to use for your section.

## Before you begin

**Sign in:** Go to [arcgis.com](https://www.arcgis.com) and sign in with the **ArcGIS organizational account** provided for this course. Do not use a free public account — free accounts cannot run most Analysis tools.

> **Warning — credits and privileges:** Running Analysis tools in ArcGIS Online consumes **credits** from your organization's account. Path A (Summarize Within) uses a modest number. Path B (raster Zonal Statistics) uses substantially more and requires **Raster Analysis privileges** or an **ArcGIS Image for ArcGIS Online** license. Use only the path your instructor authorizes. If you are unsure of your privilege level, ask before running any analysis tool.

**Time estimate:** 75–100 minutes for Path A. Add 20–30 minutes for Path B if authorized.

**Prerequisites:** You should be comfortable adding layers, opening the Styles pane, and applying symbology (covered in Labs 1–2). No prior raster experience is required for Path A.

---

## Part 1: Create a Web Map and Add Data

1. From [arcgis.com](https://www.arcgis.com), click the app launcher (grid icon, top right) and choose **Map Viewer**.
2. A new, untitled map opens. Click **Save and open** (disk icon, top left) → **Save as**. Enter a title such as `Lab4c_HeatSVI_Philadelphia`, add tags (`heat`, `SVI`, `Philadelphia`, `Lab4c`), and a brief summary. Click **Save**.
3. In the **Contents** toolbar (left side), click **Add** → **Browse layers**.
4. In the search bar, change the scope from **My Content** to **Living Atlas**. Search for `Social Vulnerability Index CDC`. Locate the **CDC/ATSDR Social Vulnerability Index (2022)** census tract layer. Click **Add**. Close the browse pane.
5. Click **Add** → **Browse layers** again. Search for your heat layer as directed by your instructor (for example: `heat severity Philadelphia` or use the instructor-shared item ID). Add that layer. Close the browse pane.
6. In the **Layers** list (Contents toolbar), drag the SVI tracts layer above the heat layer so tracts appear on top.

*[Figure: Map Viewer Contents toolbar showing two layers — heat layer beneath, SVI tracts above — over a basemap of Philadelphia]*

> **Note:** If your instructor has shared a specific ArcGIS Online item for the heat data, click **Add** → **Browse layers** → **My Organization** and search by the provided name or item ID. If the heat data is a CSV of point samples (e.g., Landsat centroid temperatures), choose **Add layer from file** and upload the CSV; you will summarize those points within tract polygons in Part 2.

---

## Part 2: Attach a Mean Heat Value to Each Tract (Path A — Feature-Based, Primary)

This path works with the heat layer as a feature layer (polygon, point, or raster-to-feature) rather than raw raster pixels. It uses the **Summarize Within** tool, which does not require raster-analysis privileges.

> **Note — why feature-based?** Standard ArcGIS Online accounts cannot run pixel-level raster zonal statistics without Raster Analysis privileges. Path A reaches the same public health answer — mean heat by tract — using the Analysis tools available to all publisher-level accounts. See Path B at the end of this part if your instructor has authorized raster analysis.

### Inspect the heat layer

1. Click the heat layer in the **Contents** toolbar to select it. Click **Table** (Contents toolbar) to open the attribute table. Identify the field that holds the heat or temperature value — your instructor will confirm the field name (for example, `LST_MEAN`, `HeatIndex`, or `Severity`).
2. Close the table. Click **Styles** (right toolbar) and apply a **Counts and Amounts (Color)** drawing style on the heat field to confirm values are sensible across Philadelphia.

### Run Summarize Within

3. Click **Analysis** (toolbar, top right area). In the Analysis pane, search for **Summarize Within**. Click the tool to open it.

4. Set tool parameters:

| Parameter | Value |
|---|---|
| Input area layer (polygons to summarize within) | SVI census tracts layer |
| Input summary layer (layer with heat values) | Heat layer (features or points) |
| Add statistics for | Heat value field (e.g., `LST_MEAN`); statistic type **Mean** |
| Output layer name | `phl_tracts_heat_mean` |

5. Review the credit estimate shown in the tool pane.

> **Warning:** The credit estimate appears before you run the tool. Note the number and confirm with your instructor before clicking **Run** if it seems unexpectedly high.

6. Click **Run**. The tool may take 1–3 minutes. A new layer `phl_tracts_heat_mean` is added to the map containing one record per tract with a field for the mean heat value (typically named `mean_LST_MEAN` or similar — check the table to confirm).

*[Figure: Summarize Within tool pane filled in with tracts as the area layer and heat points/features as the summary layer]*

7. Open the **Table** of `phl_tracts_heat_mean` and scroll to confirm the mean heat field is populated and values are plausible (e.g., °C in the low-to-mid 30s for summer Philadelphia LST, or a heat severity index between 0 and 1 depending on the layer).

> **Public health note:** Land-surface temperature (LST) measured by satellite is a surface thermal emission — not air temperature, not personal heat exposure. LST reflects impervious surfaces, dark rooftops, and sparse tree cover. It is a useful spatial indicator for prioritizing neighborhoods but should not be interpreted as what residents actually experience outdoors or indoors. Zonal and areal summaries like this one are the most practical raster or feature operation in public health GIS: the same workflow applies to PM2.5 by census tract, NDVI (tree canopy) by neighborhood, or impervious surface by block group.

---

### Path B (Optional — Raster Zonal Statistics): Requires Raster Analysis Privileges

> **Note:** Skip this section unless your instructor has confirmed your organizational account has **Raster Analysis** or **ArcGIS Image for ArcGIS Online** privileges. Running this path without the correct license will return an error and may still consume credits.

If authorized, proceed as follows:

1. Add the Landsat LST imagery layer to your map (Living Atlas or instructor-provided hosted imagery service URL via **Add** → **Add layer from URL**).
2. Click **Analysis** → search for **Zonal Statistics as Table** (under Raster Analysis tools). If the tool does not appear, your account lacks the required privilege — stop here and use Path A.
3. Set parameters:

| Parameter | Value |
|---|---|
| Input zone layer | SVI census tracts |
| Zone field | `GEOID` (11-digit tract FIPS code) |
| Input value raster | LST imagery layer |
| Statistics to calculate | Mean (optionally also Min, Max) |
| Output table name | `lst_zonal_stats` |

4. Click **Run**. This creates a standalone table saved to **My Content**.
5. Open the SVI tracts layer table. Click **Join Features** (Analysis pane) or use **Fields** → the table cannot be directly joined in the layer table view; instead, open the output table in **My Content**, then use **Join Features** to join `lst_zonal_stats` to the tracts layer on `GEOID`.
6. Confirm the `MEAN` field is present and populated in the joined tracts layer before proceeding to Part 3.

---

## Part 3: Symbolize Tracts by Mean Heat Value

1. In the **Layers** list, click `phl_tracts_heat_mean` (Path A) or the joined SVI tracts layer (Path B) to select it.
2. Click **Styles** in the right toolbar.
3. Click **+ Field**, choose the mean heat field produced in Part 2.
4. Under **Pick a drawing style**, select **Counts and Amounts (Color)**. Click **Style options**.
5. Set the classification method to **Natural Breaks** and **5 classes**. Choose a sequential color ramp from light yellow to deep red or orange (high values = hottest tracts = darkest color). Click **Done**.
6. Click **Edit layer style** → scroll to **Legend** and edit class labels to show approximate temperature ranges or heat-severity labels, as appropriate for your data.

*[Figure: Philadelphia census tracts choropleth, graduated yellow-to-red color, 5 natural-breaks classes by mean heat value; hottest tracts in dark red concentrated in dense urban areas]*

> **Public health note:** Heat is not the same as heat exposure, and heat exposure is not the same as heat morbidity. This map shows a surface thermal proxy. No health outcome data (emergency department visits, heat-related deaths) are present. Interpreting high-LST tracts as the most dangerous confuses a prioritization tool with a causal finding.

---

## Part 4: Classify SVI and Compare with Heat

### Display SVI quartiles

1. Click the SVI tracts layer in the **Layers** list. Click **Styles** (right toolbar).
2. Click **+ Field** and choose `RPL_THEMES` (the SVI overall percentile, 0–1 scale; values near 1 = highest vulnerability).
3. Select **Counts and Amounts (Color)**. Click **Style options**.
4. Change classification method to **Quantile** and **4 classes** (each class = one quartile). Apply a light-to-dark purple ramp: Q4 (most vulnerable) in dark purple, Q1 in pale lavender. Click **Done**.

*[Figure: SVI quartile choropleth, light-to-dark purple, 4 quantile classes; darkest tracts = highest social vulnerability]*

### Compare the two layers

5. Toggle the heat layer and the SVI layer on and off using the visibility toggle (eye icon) next to each layer in the **Layers** list. Look for tracts that are high on both variables — dark red on the heat map AND would be dark purple if SVI were shown.
6. Reduce the **Transparency** of the SVI tracts layer (right toolbar → **Effects → Transparency**) to approximately 40% so the heat layer shows through. This creates a rough visual overlay without a formal bivariate display.
7. Click on individual tracts with the **Pop-up** tool to read both the mean heat value and `RPL_THEMES` for the same tract. Record at least three tracts (by GEOID or neighborhood name) that appear in the hottest heat class and SVI Q4.

> **Public health note:** Spatial co-occurrence of high LST and high SVI is not evidence of worse health outcomes — it is a signal for prioritization. A full heat vulnerability assessment would add air-conditioning prevalence, cooling center proximity, age structure (elderly population share), and emergency department visits for heat illness. This comparison is an early-stage screening tool.

---

## Part 5: Finalize and Share the Web Map

1. Turn off any layers not needed for the final display. Confirm the map is centered on Philadelphia at a readable scale (roughly 1:150,000).
2. Click **Basemap** (Contents toolbar) and choose **Light Gray Canvas** or **Human Geography** for a clean background that does not compete with your color ramps.
3. Click **Save and open** → **Save** to save the current state.
4. Click **Share** (top right). Set the sharing level to **Your organization** (or **Everyone** if your instructor directs). Copy the item URL.
5. To capture a map image: click the **Print** tool if your instructor has configured it in the organization, or take a full-screen screenshot of the map view with your layers visible. Save the image as `Lab4c_HeatSVI_[YourName].png`.

*[Figure: Share dialog showing sharing level set to organization; item URL highlighted for copying]*

---

### (Optional Bonus) Bivariate Display

ArcGIS Online Map Viewer does not have a dedicated bivariate renderer. Two practical approaches:

**Approach 1 — Two-variable Arcade expression.** Open the heat tracts layer → **Fields** (right toolbar) → **Add field** → name it `BVCLASS`, type Text. Click **Calculate** and write an Arcade expression that assigns a code combining the heat class (1–5) and SVI quartile (1–4), for example `"H5_Q4"` for the hottest, most vulnerable category. Then symbolize by **Types (Unique symbols)** on `BVCLASS` and manually assign colors in a 2×2 or 5×4 grid where the darkest alarming color marks the highest heat + highest vulnerability combination.

**Approach 2 — ArcGIS Instant Apps.** Save and share your web map, then open it in an **Instant App** that supports side-by-side or swipe comparison (e.g., the **Swipe** template). This lets viewers toggle between the heat map and the SVI map interactively without a custom renderer.

*[Figure: Optional bivariate or swipe Instant App showing heat layer and SVI layer side by side for Philadelphia]*

---

## Deliverables

1. **Shared web map URL.** The map must include the heat-by-tract layer and the SVI quartile layer, both symbolized as described. Paste the URL into your lab submission.

2. **Exported map image** (`Lab4c_HeatSVI_[YourName].png`). The image should show Philadelphia tracts with at least one thematic layer visible, a legend, and a title. Capture this as a screenshot from Map Viewer or via the Print tool.

3. **Written interpretation (~200 words).** Identify 2–3 tracts (by GEOID or neighborhood name) that appear in both the highest heat class and SVI Q4. Describe the spatial pattern. State explicitly what this analysis cannot show — specifically that LST is a surface proxy, not air temperature or personal heat exposure, and that no morbidity data are included. See the AI task below for a scaffolded drafting process.

4. *(Bonus)* **Bivariate display** — screenshot or Instant App URL, with a brief description of how you constructed it.

Label any AI-assisted output clearly (see below).

---

## Using AI in this lab

### Suggested task: interpret the overlay map and draft an equity summary

After completing Parts 3 and 4, use the following prompt in Claude (claude.ai) or another AI assistant. Do not include any patient records, tract-level health counts, or PHI.

> "I have mapped Philadelphia census tracts by two variables: (1) mean summer land-surface temperature as a heat proxy, and (2) CDC/ATSDR Social Vulnerability Index quartile. High-heat, high-SVI tracts appear concentrated in [describe the pattern you observed — for example, 'North Philadelphia and parts of Kensington']. Draft a 200-word equity-framed summary suitable for a neighborhood advisory board presentation."

**Critical review task.** Read the AI draft carefully and flag every sentence where the AI:
- Describes LST as heat *exposure* or predicts *illness* (LST is a surface proxy, not personal exposure)
- Implies a health outcome — emergency visits, hospitalizations, deaths — absent from this dataset
- Makes causal claims that the two spatial layers you analyzed do not support

Revise the draft to correct those overstatements. Include two sentences in your written deliverable explaining what you changed and why.

### AI guardrails (always apply)

- AI assists with **scaffolding** (draft text, example Arcade expressions), **explaining** (what zonal summaries mean, how SVI is constructed), and **critiquing** (your map design or written interpretation).
- AI does **not** make analytic decisions, choose datasets, set classification breaks, or interpret maps it cannot see.
- Never paste PHI or individually identifiable data into a public AI system.
- Label all AI-assisted output clearly: for example, "AI-assisted draft, reviewed and revised by [your name]."

---

## ArcGIS Pro equivalent

The ArcGIS Pro version of this lab (Lab 4c) covers the same public health question using desktop raster tools.

| ArcGIS Online (this lab) | ArcGIS Pro equivalent |
|---|---|
| Add Living Atlas LST/heat feature layer | Add instructor-provided `phl_LST_summer.tif` raster |
| **Summarize Within** → mean heat per tract | **Zonal Statistics as Table** (Spatial Analyst) → MEAN per tract |
| **Join Features** (or table join on `GEOID`) | **Add Join** in Contents pane on `GEOID` |
| **Counts and Amounts (Color)**, 5 Natural Breaks classes | **Graduated Colors** symbology, 5 Natural Breaks classes |
| Quantile classification on `RPL_THEMES` | Quantile classification on `RPL_THEMES` |
| Share web map; screenshot for deliverable | Export map layout as PDF |
| No built-in bivariate renderer; use Arcade + Unique Symbols or Instant Apps | Manual bivariate approximation via Calculate Field + Unique Values renderer |

**Key difference:** ArcGIS Online reprojects layers to Web Mercator on the fly; you do not manage coordinate systems manually. Pro requires that all layers share a common coordinate reference system and that the Spatial Analyst extension is licensed. The CRS-mismatch errors common in desktop workflows do not arise in ArcGIS Online because the platform handles reprojection automatically — but this also means you have less direct control over the projection used in spatial calculations.

---

## Check your understanding

1. In Path A, **Summarize Within** produces one record per census tract with a mean heat value. What does "mean" actually represent here — and what information is lost compared to having the full distribution of heat values within each tract?

2. A colleague argues that the census tracts with the highest mean LST are the most dangerous neighborhoods for heat-related illness. What is wrong with this claim? Name at least two additional data sources you would need to assess actual heat health risk.

3. Why does Path B (raster Zonal Statistics) require special privileges in ArcGIS Online while Path A (Summarize Within) does not? What is different about how each tool processes data on Esri's servers?

4. You classified `RPL_THEMES` using **Quantile** (4 classes) rather than Natural Breaks or Equal Interval. What does quantile classification guarantee about the number of tracts in each class, and why might that be preferable when comparing across the SVI distribution?

5. ArcGIS Online reprojects layers automatically when you add them to a map. How does this differ from the workflow in ArcGIS Pro, and what is one potential disadvantage of not having direct control over the display projection?

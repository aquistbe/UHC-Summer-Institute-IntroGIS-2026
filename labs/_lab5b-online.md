## Public health question

Extreme heat is the leading weather-related cause of death in the United States, and its burden falls unevenly across neighborhoods. Communities with high impervious cover, low tree canopy, elevated land-surface temperatures, and concentrated social vulnerability face compounding risks. This lab asks: which Philadelphia census tracts carry the highest combined heat-health risk, and how do we communicate that finding responsibly? By assembling tract-level environmental and social layers into a composite vulnerability index, you can direct heat-mitigation resources — cooling centers, tree planting, outreach — toward the places that need them most.

---

## What you will learn

- Assemble tract-level inputs from the Living Atlas and instructor-shared layers onto a single feature layer using **Join Features** and **Enrich Layer**
- Normalize each input variable to a common 0–1 scale and compute a weighted composite index using **Calculate** with an Arcade expression
- Run a sensitivity check by reweighting inputs and comparing spatial patterns
- Style a choropleth with a colorblind-safe sequential color ramp and compare **Quantile** vs. **Natural Breaks** classification
- Share a web map and embed it in an **ArcGIS StoryMap** or **ArcGIS Dashboard** as a public-facing communication product
- Apply ethical framing to vulnerability maps intended for community or media audiences

---

## What you will produce

1. A saved, shared web map showing the composite heat-health vulnerability index by Philadelphia census tract (primary weighting)
2. A second field on the same layer showing the sensitivity-check composite (alternative weights), with a visual comparison
3. A public-facing communication product: either an **ArcGIS StoryMap** (narrative + map) or an **ArcGIS Dashboard** (indicators + map) — one of the two, your choice
4. A 150-word written methods note stating which inputs you used, how you normalized them, and the weights for both the primary and sensitivity runs
5. AI-assisted accessibility alt-text and a map critique response, labeled and annotated (see **Using AI in this lab**)

---

## Data

| Layer name | Description | Source |
|---|---|---|
| `PHL_Tracts_HeatSummary` | Philadelphia census tracts with mean summer LST (°C) per tract, summarized in Lab 4c | Instructor-shared ArcGIS Online item |
| CDC SVI 2022 — Pennsylvania | CDC/ATSDR Social Vulnerability Index 2022, tract level; field `RPL_THEMES` (0–1 overall rank) | Living Atlas of the World |
| NLCD Land Cover / Tree Canopy (Living Atlas) | National Land Cover Database tree canopy or impervious-surface summary at the tract level; use **Enrich Layer** or the Living Atlas NLCD-derived tract layer | Living Atlas of the World / Esri Enrich |
| Philadelphia County boundary | Administrative boundary for clip and context | Living Atlas — USA Counties |

> **Note:** This lab does not use raster inputs directly. The LST raster from Lab 4c was summarized to tracts in that lab; the result is the instructor-shared `PHL_Tracts_HeatSummary` layer. If that layer is not yet available, ask your instructor. The approach here — a vector composite index on tracts — replaces the raster **Weighted Overlay** workflow used in the ArcGIS Pro version of this lab. ArcGIS Online's standard Map Viewer does not provide a simple weighted raster overlay without **ArcGIS Image for ArcGIS Online** raster-analysis privileges (which consume additional credits). The tract-based approach is epidemiologically appropriate: most heat-health interventions target neighborhoods, and census tracts are a natural administrative unit for resource allocation.

---

## Before you begin

**Prerequisites:** Labs 1–5a, including Lab 4c (LST summarized to tracts). You need access to the instructor-shared `PHL_Tracts_HeatSummary` item in your ArcGIS organizational account.

**Sign in:** Go to [arcgis.com](https://arcgis.com) and sign in with your **ArcGIS organizational account** provided by the course. A free public account cannot run Analysis tools.

> **Warning — credits and privileges:** Running **Join Features**, **Enrich Layer**, and any other Analysis tool consumes **credits** from your organization's account and requires **Publisher** or **Analysis** privileges. Do not run these tools repeatedly with large datasets. Plan your parameters before clicking **Run**. If you are unsure of your privilege level, ask your instructor before proceeding.

**Time estimate:** 2.5–3 hours, including the StoryMap or Dashboard step.

---

## Part 1: Load and inspect the base tract layer

1. Sign in to [arcgis.com](https://arcgis.com). From the app launcher (the grid icon at top right), choose **Map Viewer**.
2. In the left toolbar, click **Add** → **Browse layers** → **My Organization**. Search for `PHL_Tracts_HeatSummary`. Click the **+** icon to add it to the map.
3. In the left toolbar, click **Tables** (table icon). The attribute table opens at the bottom. Locate the field that holds mean summer LST per tract. It is likely named `MEAN_LST` or similar — confirm with your instructor. Note the approximate range of values (minimum and maximum).
4. Close the table. Click the layer name in the **Contents** pane to select it, then click **Fields** in the right toolbar. Scroll through the available fields. You need at minimum: a tract identifier (`GEOID` or `FIPS`), and the LST summary field.

*[Figure: Map Viewer showing PHL_Tracts_HeatSummary layer loaded, attribute table open at bottom with GEOID and MEAN_LST columns visible]*

---

## Part 2: Join the CDC SVI layer

You will join the CDC SVI percentile rank (`RPL_THEMES`) onto your tract layer so all inputs live on one feature layer.

1. In the left toolbar, click **Add** → **Browse layers** → **Living Atlas**. Search for `CDC Social Vulnerability Index 2022`. Add the Pennsylvania or national layer to the map. It will appear as a separate layer in the **Contents** pane.
2. In the left toolbar, click **Analysis** (the tools icon). In the analysis tools pane, search for **Join Features**.
3. Set **Join Features** parameters:

   | Parameter | Value |
   |---|---|
   | Target layer | `PHL_Tracts_HeatSummary` |
   | Join layer | CDC SVI 2022 |
   | Join type | One to one |
   | Join condition | Spatial — target contains join feature centroid, OR attribute match on `FIPS` / `GEOID` if both layers share that field |
   | Fields to join | `RPL_THEMES` only (to minimize output size) |
   | Output layer name | `PHL_Tracts_Joined` |

4. Click **Run**.

> **Warning:** This tool consumes credits. Confirm your parameters before clicking **Run**. If the SVI layer covers all of Pennsylvania, the spatial join will still work correctly because it matches only within the Philadelphia tracts in your target layer.

*[Figure: Join Features tool pane showing target layer, join layer, and output name filled in]*

---

## Part 3: Add a tree canopy or impervious surface variable

You need a third environmental input. Use **Enrich Layer** if your organization has Enrich credits, or add a Living Atlas NLCD-derived tract layer and join it as in Part 2.

### Option A — Enrich Layer (if available)

1. Select `PHL_Tracts_Joined` in the Contents pane. Click **Analysis** → search **Enrich Layer**.
2. Click **Select variables**. Under the Environmental or Land Cover category, find a variable related to tree canopy coverage or impervious surface (Esri's enrichment data catalog includes NLCD-derived land cover summaries).
3. Output layer name: `PHL_Tracts_Enriched`. Click **Run**.
4. After the tool completes, open **Tables** and confirm a tree canopy or impervious variable field was added.

> **Warning:** Enrich Layer can consume significant credits depending on the variables selected and the number of polygons enriched. Select only the one variable you need.

### Option B — Join a Living Atlas NLCD layer

1. In **Add** → **Browse layers** → **Living Atlas**, search for `NLCD Tree Canopy` or `USA NLCD Tree Canopy`. Add the layer.
2. Run **Join Features** again: target = `PHL_Tracts_Joined`, join = NLCD tract layer. Join on a shared FIPS/GEOID field (attribute join) or spatial centroid. Join the tree canopy percentage field only. Output: `PHL_Tracts_Enriched`.

> **Note:** Either option produces the same result: a single tract layer with LST, SVI, and tree canopy fields. Rename your final layer `PHL_Tracts_Enriched` regardless of which option you used. Note in your methods write-up which option you used.

---

## Part 4: Normalize inputs and compute the composite index

With all inputs on one layer, you will normalize each to a 0–1 scale and calculate a weighted composite using Arcade in the **Calculate** field tool.

### 4a — Add normalized fields

1. In the Contents pane, select `PHL_Tracts_Enriched`. In the right toolbar, click **Fields**.
2. Click **+ Add field**. Name: `LST_norm`, Type: **Double**. Click **Save** (the checkmark). Repeat to add `SVI_norm` (Double) and `CANOPY_norm` (Double).

### 4b — Calculate LST_norm

Min-max normalization scales each value to 0–1. A value of 1 = highest LST (highest heat exposure). You need the observed minimum and maximum LST across all tracts. Read them from the table in Part 1 or use the **Statistics** panel.

1. Click the `LST_norm` field header → **Calculate**.
2. In the Arcade expression box, enter:

   ```
   ($feature.MEAN_LST - 28.5) / (42.1 - 28.5)
   ```

   Replace `28.5` with your observed minimum and `42.1` with your observed maximum. This maps the coldest tract to 0 and the hottest tract to 1.

3. Click **OK**.

> **Note:** Arcade expressions in **Calculate** do not automatically compute dataset-wide statistics like min and max — you must supply those numbers manually from the table or field statistics. Record the values you use; they must appear in your methods write-up.

### 4c — Calculate SVI_norm

`RPL_THEMES` is already a 0–1 percentile rank, so normalization is simple. A value of 1 = highest social vulnerability.

1. Click `SVI_norm` → **Calculate**. Expression:

   ```
   $feature.RPL_THEMES
   ```

2. Click **OK**.

> **Note:** SVI values of -999 indicate missing data. Filter those records out before calculating if any tracts show -999: in the right toolbar click **Filter**, set `RPL_THEMES` does not equal `-999`, apply the filter, then run Calculate only on filtered records. Remove the filter afterward.

### 4d — Calculate CANOPY_norm

High tree canopy = lower heat-health risk. To align direction so that 1 = highest risk, invert the canopy variable.

1. Click `CANOPY_norm` → **Calculate**. Expression:

   ```
   1 - (($feature.TREE_CANOPY_PCT - 0) / (100 - 0))
   ```

   Replace `TREE_CANOPY_PCT` with the actual field name from your enriched layer. This maps 0% canopy to 1 (highest risk) and 100% canopy to 0 (lowest risk).

2. Click **OK**.

### 4e — Add the composite field and calculate (primary weights)

**Primary weights:** LST 40%, SVI 30%, Tree Canopy 30%. These weights reflect that land-surface temperature is the most direct driver of acute heat exposure; canopy and social vulnerability are important modifiers. These weights are a deliberate analytic choice — document them.

1. Add a field: `HVI_primary`, Type **Double**.
2. Click `HVI_primary` → **Calculate**. Expression:

   ```
   ($feature.LST_norm * 0.40) + ($feature.SVI_norm * 0.30) + ($feature.CANOPY_norm * 0.30)
   ```

3. Click **OK**. Values will range from 0 to 1, where higher values indicate greater heat-health vulnerability.

> **Warning:** Weights are a value judgment. There is no objectively correct weighting for a composite vulnerability index. The choice of 40/30/30 is defensible but not uniquely correct. The sensitivity check in Part 5 tests how much your conclusions depend on this choice. Document both runs and present the uncertainty honestly.

*[Figure: Fields panel showing HVI_primary field selected and Arcade expression entered in the Calculate dialog]*

---

## Part 5: Sensitivity check with alternative weights

A single weighting scheme is one analytic choice among many. Running a second scenario reveals whether your conclusions are robust.

**Alternative weights:** LST 20%, SVI 60%, Tree Canopy 20%. This scenario treats social vulnerability as the dominant driver, consistent with a social determinants of health framework.

1. Add a field: `HVI_sensitivity`, Type **Double**.
2. Click `HVI_sensitivity` → **Calculate**. Expression:

   ```
   ($feature.LST_norm * 0.20) + ($feature.SVI_norm * 0.60) + ($feature.CANOPY_norm * 0.20)
   ```

3. Click **OK**.
4. Open the **Table**. Sort by `HVI_primary` descending and note the top 10 tracts. Then sort by `HVI_sensitivity` descending and note the top 10. Are the same tracts in both lists?

> **Public health note:** If the primary and sensitivity runs identify the same high-risk tracts, your index is robust to this weight change. If they diverge substantially, acknowledge the uncertainty in your communication product and avoid presenting the map as a definitive ranking. Showing only the run that confirms a preferred narrative is analytic bias.

---

## Part 6: Style the composite map

1. In the Contents pane, select `PHL_Tracts_Enriched`. In the right toolbar, click **Styles**.
2. Under **Pick a field to show**, choose `HVI_primary`.
3. In **Pick a style**, select **Counts and Amounts (Color)**.
4. Click **Style options**. Under **Theme**, set classification method to **Quantile** with **5 classes**. Note how the class breaks are assigned.
5. Under **Color ramp**, choose a colorblind-safe sequential ramp. **Yellow-Orange-Red** (ColorBrewer) or **Viridis** are good choices. Apply so that low values (lower vulnerability) appear light and high values (higher vulnerability) appear dark red or purple.
6. Click **Done**.
7. Repeat steps 1–6 on a duplicate of the layer styled on `HVI_sensitivity` to enable a visual comparison. Toggle between the two styled layers.

> **Note:** Try switching classification from **Quantile** to **Natural Breaks** and observe how the map changes. Natural breaks emphasizes data-driven groupings; quantile guarantees equal tract counts per class. Note which you prefer and state the reason in your methods write-up.

> **Public health note:** Colorblind-safe ramps matter for any map shown to a public audience. Approximately 8% of men and 0.5% of women have red-green color vision deficiency. Red-green diverging ramps (red = bad, green = good) should be avoided for vulnerability maps.

*[Figure: Styles panel showing HVI_primary field, Counts and Amounts style, Quantile 5-class, Yellow-Orange-Red ramp applied to Philadelphia tracts]*

---

## Part 7: Save and share the web map

1. Click **Save** (the floppy disk icon at the top of the screen). In the **Save map** dialog: Title: `Heat-Health Vulnerability Index — Philadelphia PA`; Tags: `heat`, `public health`, `Philadelphia`, `vulnerability`; Summary: `Composite tract-level heat-health vulnerability index combining land-surface temperature, CDC Social Vulnerability Index, and tree canopy cover. Intro GIS for Public Health capstone.`
2. Click **Save map**.
3. Click **Share map** (the share icon). Under **Who can view this map**, choose your organization (or as directed by your instructor). Click **Save**.
4. Copy the item URL from your browser's address bar or from **My Content**. You will embed this in your communication product in Part 8.

---

## Part 8: Build a public-facing communication product

ArcGIS Online excels at sharing spatial findings with non-GIS audiences. Choose one of the two options below. Both are accessible from the Esri app launcher.

### Option A — ArcGIS StoryMap (recommended for narrative framing)

A StoryMap pairs your map with text, photos, and context in a scrollable web page — ideal for communicating with community members, policymakers, or media.

1. From the app launcher, choose **StoryMaps**. Click **+ New story** → **Start from scratch**.
2. Add a **Title** block: `Heat-Health Vulnerability in Philadelphia: Which Neighborhoods Need Cooling Resources Most?`
3. Add a **Text** block of 2–3 sentences stating the public health question in plain language. Avoid jargon. Avoid language that stigmatizes residents (see the public health note below).
4. Add a **Map** block: click **+** → **Map** → **Select a map from ArcGIS Online**. Search for your saved `Heat-Health Vulnerability Index — Philadelphia PA` web map. Select it. The interactive map embeds in the story.
5. Add a **Text** block after the map explaining what colors represent and one key finding from the spatial pattern (for example, which cluster of tracts consistently appears in the highest-risk class across both weighting scenarios).
6. Add a **Text** block for data sources and a brief methods note (two to three sentences maximum for a public audience).
7. Click **Publish** → **Everyone** or **Organization** as instructed. Copy the StoryMap URL for your deliverables.

*[Figure: StoryMaps editor with title block, embedded web map block, and text block visible in the editing panel]*

> **Public health note:** Language matters. "High-risk tracts" describes structural conditions — not the people living in those tracts. Avoid phrasing like "residents who are at risk" when the risk drivers are physical environment and historical disinvestment. Label what drives the score (heat, lack of canopy, social vulnerability) rather than naming neighborhoods as dangerous. Acknowledge in your story that the index is a tool to guide resource allocation, not a definitive measure of individual health risk. Stigmatizing maps can harm communities and undermine trust in public health agencies.

### Option B — ArcGIS Dashboard (recommended for indicator-style reporting)

A Dashboard presents a map alongside summary statistics — appropriate for internal reporting or public health department communication products.

1. From the app launcher, choose **Dashboards**. Click **+ New dashboard**.
2. Click the **+** icon in the canvas → **Map**. Select your saved web map. Resize the map panel to occupy most of the canvas.
3. Click **+** → **Indicator**. Set the data source to `PHL_Tracts_Enriched`. Statistic: **Maximum** of `HVI_primary`. Label: `Highest Vulnerability Score`. Add a second Indicator for **Average** of `HVI_primary`.
4. Click **+** → **Serial Chart** (bar chart). Data source: `PHL_Tracts_Enriched`, sorted by `HVI_primary` descending, showing the top 15 tracts. Add tract name or GEOID as the category field.
5. Save the dashboard. Share it with your organization or as directed by your instructor. Copy the URL.

*[Figure: Dashboard editor showing the embedded map panel on the left, two indicator widgets at top right, and a bar chart of top-scoring tracts at bottom right]*

---

## Deliverables

Submit all of the following to your instructor's LMS drop folder by the end of Day 5:

| # | Item | Notes |
|---|---|---|
| 1 | Web map URL | Shared `Heat-Health Vulnerability Index — Philadelphia PA` map; confirm it is accessible to your organization |
| 2 | StoryMap URL or Dashboard URL | Whichever communication product you built in Part 8 |
| 3 | Screenshot of the styled `HVI_primary` map | Export from the Print tool or take a screen capture at full resolution; include the legend |
| 4 | Written methods note | 150 words max; state inputs used, normalization approach and min/max values used, weights for primary and sensitivity runs, and classification method chosen |
| 5 | Sensitivity comparison note | 2–3 sentences: which tracts appeared in the top quintile in the primary run but not the sensitivity run (or vice versa), and what that implies for analytic confidence |
| 6 | AI-assisted map alt-text | One paragraph; label which sentences are AI-drafted and what you changed |
| 7 | AI map critique response | List the critique points identified; note which you acted on and which you did not, with a brief reason |

---

## Using AI in this lab

AI assistance is permitted for two specific, bounded tasks.

**Task A — Accessibility alt-text.** Accessibility requires descriptive alt-text for maps posted online or in reports. Share a screenshot of your styled `HVI_primary` map with Claude and prompt: *"Write one paragraph of accessibility alt-text for this heat-health vulnerability map of Philadelphia census tracts. Describe the geographic pattern, the color ramp, the classification method, and the key finding. Aim for 75–100 words."* Edit any factual errors. Label the output: `[AI-drafted, edited by [Your Name]]`.

**Task B — Map critique.** Prompt Claude: *"Critique this map for: (1) color choice and colorblind accessibility, (2) appropriateness of the classification method (quantile vs. natural breaks), and (3) ethical framing — does the title, legend, or StoryMap text risk stigmatizing neighborhoods? Suggest specific improvements."* In your deliverables, list each critique point and state whether you acted on it and why.

**Guardrails that apply to this course:**

- AI helps with scaffolding, explaining, and critiquing. It does not make analytic decisions, choose datasets, select weights, or interpret spatial patterns in the data.
- Never paste patient-level, address-level, or otherwise identifiable public health data into a public LLM.
- Label all AI-assisted output in your submitted work. Unlabeled AI output is an academic integrity violation.
- Verify every AI-generated factual claim before including it in your submission.

---

## ArcGIS Pro equivalent

The Pro version of this lab (Lab 5b: Cartographic Modeling Capstone) uses a fully raster-based workflow. The table below maps each ArcGIS Online step to the corresponding Pro tool.

| ArcGIS Online step | ArcGIS Pro equivalent |
|---|---|
| Join Features + Enrich Layer (Parts 2–3) | Load raster inputs directly; **Polygon to Raster** to convert SVI vector to raster |
| Calculate (Arcade, min-max normalization) | **Reclassify** — rescales each raster input to a 1–5 ordinal scale using quantile or equal-interval breaks |
| `HVI_primary` weighted sum (Arcade) | **Weighted Overlay** — applies percentage weights to reclassified rasters and produces a composite raster |
| Second Calculate field `HVI_sensitivity` | Second **Weighted Overlay** run with alternative weights |
| Styles → Counts and Amounts (Color) | **Symbology** pane → Classify → color ramp |
| StoryMap or Dashboard | **Layout** in Pro → Export to PDF/PNG; or **ArcGIS Online Share** for web communication |

**Key difference:** The ArcGIS Online approach operates on tract polygons (vector composite index); the Pro approach operates on 30-meter raster cells. The raster approach captures within-tract variation in LST and canopy that the tract-level average smooths over. For intervention planning at the neighborhood level, both approaches yield comparable priority areas. The desktop Lab 5b exists for users who need raster precision.

---

## Check your understanding

1. You inverted the tree canopy variable so that high canopy coverage maps to a low vulnerability score (near 0). Explain in one sentence why this inversion is necessary to make all three inputs point in the same direction.
2. In the primary weights, LST received 40% weight and SVI received 30%. Describe one alternative weighting scheme you could justify from a social determinants of health perspective and state the reasoning.
3. A colleague reviews your StoryMap and says: "This map shows that Kensington residents are at high risk." Explain why that framing is potentially problematic and how you would reword it.
4. You ran a sensitivity check and found that 8 of the top 10 tracts appeared in both the primary and alternative weight runs. What does this tell you about the robustness of your index? What would it mean if only 3 of 10 matched?
5. You want to add proximity to cooling centers as a fourth input. The cooling center layer is a point feature class. Describe the steps you would take in ArcGIS Online — specifically which Analysis tool you would use to create a tract-level proximity variable — and in which direction you would code the normalized value (high proximity = low or high vulnerability score).

## Public health question

Where are the most heat-vulnerable neighborhoods in Philadelphia, and how do cooling resources and flood hazards align with them? Extreme heat is the leading weather-related cause of death in the United States, and urban heat exposure is unequally distributed. This lab uses Philadelphia data to begin exploring that spatial inequality — asking whether the neighborhoods most vulnerable to heat are also best served by cooling infrastructure such as pools and spraygrounds.

---

## What you will learn

- Sign in and navigate the **Map Viewer** interface (dark left Contents toolbar; light right Settings toolbar)
- Create, save, and share a web map
- Add layers from instructor-shared org items, from a file upload (zipped shapefile), and from the **Living Atlas**
- Arrange layers and switch basemaps using the **Basemap** gallery
- Apply three drawing styles: **Location (single symbol)**, **Counts and Amounts (Color)**, and **Types (Unique symbols)**
- Classify a health index with **Equal Interval** and **Quantile** and evaluate the trade-offs
- Configure pop-ups and edit the legend
- Create bookmarks to save named views and use the **Measure** tool
- Share a web map and understand Map Viewer's layout limitations

---

## What you will produce

One saved web map with bookmarks serving as saved views, containing three layer configurations:

1. **View 1 — Flood and green infrastructure:** GSI points over FEMA 100-year flood zones with census tract outlines.
2. **View 2 — Heat vulnerability choropleth:** Tract-level HVI styled with graduated colors; bookmark zoomed to a high-vulnerability cluster.
3. **View 3 — Cooling resources:** HVI choropleth with pools and spraygrounds as point symbols.

You will submit the shared web map URL and one screenshot per view. ArcGIS Online's Map Viewer has no full print layout — no drag-and-drop north arrow, scale bar, or inset map. For polished layouts, the platform offers **Instant Apps**, **ArcGIS Dashboards**, and **ArcGIS StoryMaps**; a later lab covers those. Screenshots and the shared link are sufficient here.

---

## Data

| Layer | Description | How to add |
|---|---|---|
| `Philadelphia Census Tracts` | 2020 tract boundaries, Philadelphia County | Browse layers → My Organization (instructor-shared) |
| `GSI Public Projects` | Philadelphia Water Dept. public green stormwater infrastructure | Browse layers → My Organization (instructor-shared) |
| `100-Year Flood Zones` | FEMA 1%-annual-chance flood zone polygons | Browse layers → My Organization (instructor-shared) |
| `Heat Vulnerability by Tract` | Heat vulnerability index (`HVI`) by census tract | Browse layers → My Organization (instructor-shared) |
| `PPR Spraygrounds` | Philadelphia Parks & Recreation sprayground locations | Instructor-shared item or file upload (zipped .shp) |
| `PPR Swimming Pools` | Philadelphia Parks & Recreation swimming pool locations | Instructor-shared item or file upload (zipped .shp) |

---

## Before you begin

- **Account:** Use the **ArcGIS organizational account** your instructor provided. A free public account can create web maps but cannot run Analysis tools.
- **Credits:** This lab does not run Analysis tools, so credit use is minimal. File hosting consumes a small number of credits. Later labs use tools like **Enrich Layer** and **Create Drive-Time Areas** that consume credits quickly — check with your instructor before running any unfamiliar Analysis tool.
- **Browser:** Chrome, Firefox, Edge, or Safari. Allow pop-ups for arcgis.com if prompted.
- **Estimated time:** 2–2.5 hours.

---

## Part 1: Sign in and create a web map

1. Go to [arcgis.com](https://www.arcgis.com) and click **Sign In**. Enter your organizational credentials (or the SSO URL your instructor provides).
2. From the **app launcher** (grid icon, upper left) choose **Map Viewer**.
3. Click **Save** (disk icon) → **Save map**. Enter:

| Field | Value |
|---|---|
| Title | `Lab01_Philadelphia_YourName` |
| Tags | `Lab01`, `Philadelphia`, `heat vulnerability` |
| Summary | `Lab 1 — Philadelphia heat, flood, and cooling resources.` |

4. Click **Save**. Save frequently — Map Viewer has no auto-save.

*[Figure: Map Viewer with the dark left (Contents) toolbar and light right (Settings) toolbar; map title visible at top]*

5. In the left toolbar, click **Layers** → **Add** (plus icon) → **Browse layers**. Change the source to **My Organization**. Add `Philadelphia Census Tracts`, `GSI Public Projects`, and `100-Year Flood Zones`.
6. Drag layers into this order (top to bottom): `GSI Public Projects`, `100-Year Flood Zones`, `Philadelphia Census Tracts`.

> **Public health note:** Layer order controls drawing order. Keep points and lines above polygon layers so they remain visible.

7. Click **Basemap** in the left toolbar. Select **Light Gray Canvas** to reduce visual noise.
8. Click **Bookmarks** → **Add bookmark** → name it `Philadelphia full extent` → **Add**.

---

## Part 2: Symbology for Map View 1 — flood and green infrastructure

1. Click `Philadelphia Census Tracts` in the Layers panel. In the right toolbar, click **Styles** → **Style options** (next to **Location (single symbol)**). Set fill to **No color**, outline to medium gray, `1 px`. Click **Done** twice.
2. Click `100-Year Flood Zones`. Open **Styles** → **Style options**. Set fill to light blue at ~60% opacity, outline to the same blue, `1 px`. Click **Done** twice.

> **Public health note:** FEMA 100-year flood zones mark a 1% annual flood probability. Repeated flooding damages housing and infrastructure and concentrates health consequences — disproportionately in lower-income communities.

3. Click `GSI Public Projects`. Open **Styles** → **Style options**. Choose a dark green circle, `8 px`. Click **Done** twice.
4. With `GSI Public Projects` selected, click **Pop-ups** in the right toolbar. Verify a pop-up opens with useful field information when you click a point on the map.

*[Figure: Map showing gray tract outlines, light blue flood zones, and dark green GSI points on Light Gray Canvas]*

---

## Part 3: Heat vulnerability choropleth — Map View 2

1. Click **Add** → **Browse layers** → **My Organization**. Add `Heat Vulnerability by Tract`. Drag it to the top of the Layers panel.

> **Public health note:** The `HVI` field combines land surface temperature, impervious cover, tree canopy, poverty, and age to flag tracts with compounded heat risk. All vulnerability indices embed analytic assumptions — no index is the definitive picture.

2. Click `Heat Vulnerability by Tract`. In the right toolbar click **Styles**. Under **Pick an attribute**, select `HVI`. Click **Counts and Amounts (Color)** → **Style options**. Set:

| Parameter | Value |
|---|---|
| Classification method | Quantile |
| Number of classes | 5 |
| Color ramp | Sequential yellow-to-red (light = low; dark red = high) |

3. Edit the legend labels if needed. Click **Done** twice.

*[Figure: HVI quantile choropleth, yellow-to-red, 5 classes]*

4. Return to **Styles** → **Style options**. Switch method to **Equal interval**, keeping 5 classes. Observe how class breaks shift and how many tracts appear in the highest class. Switch back to **Quantile** when done.

> **Public health note:** Quantile places equal numbers of tracts in each class — useful for showing variation but can exaggerate moderate differences. Equal interval uses equal-width bins, more honest about magnitude but often leaves most tracts in the lowest class for a skewed index. This is an analytic decision, not a software default.

5. Pan and zoom to a cluster of dark-red tracts (North or West Philadelphia). Click **Bookmarks** → **Add bookmark** → name it `High-vulnerability cluster`. Click **Add**.

> **Note:** Map Viewer does not support inset map frames. Use bookmarks instead: `Philadelphia full extent` and `High-vulnerability cluster` let viewers navigate between the city overview and the zoomed cluster — the web equivalent of a Pro layout inset.

6. Click **Save**.

---

## Part 4: Cooling resources — Map View 3

1. Click **Add** → **Browse layers** → **My Organization**. Add `PPR Spraygrounds` and `PPR Swimming Pools`. Drag both to the top of the Layers panel. If directed to upload files, click **Add** → **Add layer from file** and browse to each zipped shapefile.
2. Click `PPR Spraygrounds`. Open **Styles** → **Style options** → **Location (single symbol)**. Choose a blue circle, `10 px`. Click **Done** twice.
3. Click `PPR Swimming Pools`. Open **Styles** → **Style options** → **Location (single symbol)**. Choose a distinct symbol — cyan square or diamond, `10 px`. Click **Done** twice.

> **Note:** Two separate layers with distinct single symbols is cleaner here than combining into one **Types (Unique symbols)** layer. Both appear as separate legend entries.

4. Click **Pop-ups** for each cooling layer to verify information appears on click.
5. Click **Legend** (left toolbar). If any entry is unclear, rename the layer via its three-dot menu → **Rename**.

*[Figure: Map with HVI choropleth, blue circle spraygrounds, and cyan diamond pools overlaid]*

---

## Part 5: Measure distance

1. In the top map toolbar, click **Measure** (ruler icon) → **Distance**.
2. Click the centroid of a dark-red tract. Click the nearest visible pool or sprayground. Double-click to end the measurement. Note the straight-line distance and units.
3. Record the distance and the two endpoints — you will report this in your deliverables.

> **Public health note:** Straight-line distance is not travel distance. A later lab uses **Create Drive-Time Areas** for realistic service areas. For now, this gives a rough proximity estimate.

---

## Part 6: Save and share

1. Click **Save**.
2. Click **Share map** → **Your organization**.

> **Warning:** Do not choose **Everyone (public)** unless directed. Organization sharing is sufficient.

3. Click **Copy link** and paste the URL into your deliverable.
4. Navigate to each bookmark and use your OS screenshot tool (Win: **Win + Shift + S**; macOS: **Cmd + Shift + 4**) to capture each view. Save as `Map1_GSI_FloodZones.png`, `Map2_HeatVulnerability.png`, and `Map3_CoolingResources.png`.

---

## Deliverables

1. **Web map URL** — shared with the organization; must include all six layers and both bookmarks.
2. **Three screenshots** — one per map view (filenames above).
3. **Classification comparison (100 words max):** Which scheme — Equal Interval or Quantile — did you choose and why? What does it emphasize and what does it obscure?
4. **Measured distance:** The straight-line distance from Part 5 and a brief description of the two endpoints.
5. **AI-assisted explanation (labeled):** The plain-language HVI explanation from Task A, labeled as AI-assisted with your edits noted.

---

## Using AI in this lab

### Task A — Explain the concept

Open Claude (claude.ai) or ChatGPT and paste:

> "In one paragraph for a public health professional without a GIS background, explain what a heat vulnerability index is, how it is typically constructed, and two methodological limitations to keep in mind when interpreting it."

Edit for accuracy if needed. Label the result **AI-assisted** in your deliverable.

### Task B — Compare classification schemes

Ask the AI:

> "I am making a choropleth web map of a heat vulnerability index for Philadelphia census tracts. Suggest two classification schemes and explain the trade-offs of each for communicating spatial health inequity."

**You** decide which scheme to use and explain that choice in Deliverable 3. The AI explains trade-offs; it does not make analytic decisions.

> **Guardrail — required reading before using AI in this course:**
> AI assists with: **scaffolding** (drafting text or structure), **explaining** (concepts and errors), and **critiquing** (your map or interpretation). AI does **not** select datasets, make analytic decisions, or interpret rare or small-area events. Never paste patient names, addresses, case IDs, or identifiable health information into a public AI tool. Label all AI-assisted output.

---

## ArcGIS Pro equivalent

| ArcGIS Online (Map Viewer) | ArcGIS Pro equivalent |
|---|---|
| Browse layers → My Organization | Add Data → file browser |
| Styles → Counts and Amounts (Color) | Symbology → Graduated Colors |
| Styles → Types (Unique symbols) | Symbology → Unique Values |
| Styles → Location (single symbol) | Symbology → Single Symbol |
| Bookmarks (saved views) | Bookmarks + inset map frame in Layout |
| Share map → copy link | Export Layout → PDF |
| No inset map | Insert → Map Frame (second map) in Layout |
| Measure (geodesic) | Measure (Euclidean; depends on CRS) |
| Projection automatic | Manage CRS explicitly; reproject as needed |

ArcGIS Online reprojects layers to Web Mercator automatically and uses geodesic math for Analysis — no Project tool needed. ArcGIS Pro gives explicit CRS control but creates more risk of coordinate-mismatch errors. The matching desktop lab is **Lab 1: Introduction to ArcGIS Pro**.

---

## Check your understanding

1. You add a polygon layer in Map Viewer and it covers all other layers. What do you do to fix this without removing the layer?
2. Why does ArcGIS Online not require you to reproject a layer before adding it, even though the map displays in Web Mercator?
3. A colleague argues Equal Interval is always more honest; another says Quantile is always better. Who is right, and under what circumstances?
4. You measure a straight-line distance of 1.4 miles from a high-vulnerability tract to the nearest pool. What does this tell you, and what does it **not** tell you about residents' actual access?
5. You need a polished map layout with a north arrow, scale bar, and legend for a report. Map Viewer cannot produce this directly. What tool would you use, and what does it produce?

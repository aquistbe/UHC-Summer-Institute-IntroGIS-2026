## Public health question

A community health team is walking four city blocks in West Philadelphia to assess pedestrian injury risk. Observations about sidewalk quality, crossing infrastructure, and traffic calming features were recorded on paper forms. To map, query, and link these records to injury outcomes, they must enter a GIS as georeferenced vector features. This lab teaches you to create a hosted feature layer, define coded-value fields, digitize points, lines, and a polygon over an imagery basemap, and calculate geometry — replicating a real field-audit workflow.

---

## What you will learn

- Create a hosted **feature layer** (point, line, and polygon sublayers) in ArcGIS Online
- Define attribute fields and set up **coded-value lists** for categorical audit variables
- Enable **editing** in layer settings and add the layer to a Map Viewer web map
- Use the Map Viewer **sketch/edit** tools to digitize the study-area polygon, street-segment lines, and observation points over an imagery basemap
- Populate attributes using the editable **pop-up form**
- Edit and reposition vertices after initial digitizing
- Inspect the layer's **attribute table**
- Calculate geometry (length, area, coordinates) with **Arcade** expressions via **Fields > Calculate**
- Understand how this hosted layer supports real field collection in **ArcGIS Field Maps**

---

## What you will produce

1. A hosted feature layer (`Lab02b_WalkingAudit`) with three sublayers: `audit_sites` (point), `street_segments` (polyline), and `study_area` (polygon), all with populated attribute tables
2. A saved, shared web map with a link submitted to the course portal
3. A screen capture showing all three layers over imagery with the `audit_sites` table visible
4. A 100-word written interpretation identifying the most hazardous location and the two coded conditions behind that judgment

---

## Data

All data are created by you during the session. No pre-existing GIS files are required.

| Layer | Description | Source |
|---|---|---|
| Esri World Imagery | High-resolution aerial imagery of Philadelphia | Built into every ArcGIS Online web map via **Basemap** |
| Walking-audit codebook (below) | Coded attribute categories for the audit | Provided in this handout |

### Walking-audit attribute codebook

| Attribute | Field name | Type | Coded values |
|---|---|---|---|
| Site ID | `site_id` | Integer | 1, 2, 3 … (sequential) |
| Lighting | `lighting` | Text | `none` · `poor` · `adequate` |
| Sidewalk condition | `sidewalk` | Text | `good` · `cracked` · `absent` |
| Crossing aids | `crossing` | Text | `none` · `marked` · `signal` · `curb_ramp` |
| Traffic calming | `traffic_calm` | Text | `none` · `speed_hump` · `bumpout` |
| Notes | `notes` | Text | Free text |
| Segment ID | `seg_id` | Integer | 1, 2, 3 … (lines only) |
| Length (ft) | `length_ft` | Double | Calculated via Arcade |
| Area (sq ft) | `area_sqft` | Double | Polygon only — calculated |
| Latitude | `lat_dd` | Double | Calculated — points only |
| Longitude | `lon_dd` | Double | Calculated — points only |

> **Public health note:** Coded-value lists prevent free-text inconsistencies — a common problem when multiple field workers contribute to the same layer. Consistent coding is essential for counting hazard exposures and linking observations to injury counts.

---

## Before you begin

**Account requirement:** You must be signed in to the **ArcGIS organizational account** provided by the course. A free public ArcGIS account cannot create hosted feature layers. Editing an existing hosted layer does **not** consume credits — but creating and publishing requires **publisher privileges** on an org account.

**Estimated time:** 2 hours. Have this handout open alongside your browser at **arcgis.com**.

---

## Part 1: AI-assisted codebook design

Before digitizing, use an AI tool to scaffold your audit protocol.

1. Open Claude (claude.ai) or another approved AI assistant in a second tab.
2. Paste this prompt:

   > *"We're auditing 4 blocks in West Philadelphia to study pedestrian injury risk. We need to capture sidewalk quality, crossing infrastructure, and traffic calming. Draft a simple field codebook — field name, description, and coded values — for a GIS attribute table."*

3. Compare the AI output to the codebook table above. Note where it matches, differs, or is incomplete.
4. In a plain-text file, write two sentences explaining one change you would make to the AI's draft and why. Save as `codebook_notes.txt`.

> **Using AI in this lab:** The AI gives a starting scaffold based on general audit knowledge. You — the public health professional — decide which categories fit your study population, IRB protocol, and local conditions. Review and refine AI-generated protocols before using them in research. Label any AI-assisted output you submit.

---

## Part 2: Create a hosted feature layer

1. Sign in to **arcgis.com**. Click **Content** → **New item** → **Feature layer** → **Define your own layer** → **Next**.
2. Add three sublayers. For each, click **Add layer**, name it, and set the geometry type:

   | Layer name | Geometry type |
   |---|---|
   | `audit_sites` | Point |
   | `street_segments` | Polyline |
   | `study_area` | Polygon |

3. Click **Next**. Enter item details: **Title** `Lab02b_WalkingAudit` · **Tags** `walking audit, pedestrian, Lab02b` · **Summary** *Built-environment audit, West Philadelphia*. Click **Save**.

The item page opens showing your three sublayers.

*[Figure: ArcGIS Online item page for Lab02b_WalkingAudit showing three sublayers with point, polyline, and polygon icons]*

---

## Part 3: Define fields and coded-value lists

### 3A. audit_sites (point)

1. On the item page, click **audit_sites** → **Fields** tab → **Add field** for each row below:

   | Field name | Type | Length |
   |---|---|---|
   | `site_id` | Integer | — |
   | `lighting` | String | 10 |
   | `sidewalk` | String | 10 |
   | `crossing` | String | 10 |
   | `traffic_calm` | String | 15 |
   | `notes` | String | 100 |
   | `lat_dd` | Double | — |
   | `lon_dd` | Double | — |

2. Click the `lighting` field name → under **List of values (domain)** click **Create list**. Add code/label pairs: `none`/None, `poor`/Poor, `adequate`/Adequate. **Save**.
3. Repeat for `sidewalk` (`good`, `cracked`, `absent`), `crossing` (`none`, `marked`, `signal`, `curb_ramp`), and `traffic_calm` (`none`, `speed_hump`, `bumpout`).

> **Note:** Coded-value lists appear as drop-down menus in the pop-up editing form — the same function as coded-value domains in ArcGIS Pro.

### 3B. street_segments (polyline)

Add the same `lighting`, `sidewalk`, `crossing`, `traffic_calm`, and `notes` fields (same lengths and lists), plus `seg_id` (Integer) and `length_ft` (Double).

### 3C. study_area (polygon)

Add one field: `area_sqft` (Double).

---

## Part 4: Enable editing and open in Map Viewer

1. On the item page, click **Settings**. Under **Feature Layer (hosted)**, enable **Editing**: check **Add**, **Delete**, and **Update attributes and geometry**. **Save**.
2. Return to **Overview** → click **Open in Map Viewer**.
3. Click **Basemap** (left toolbar) → **Imagery**.
4. In the search bar, type `40th and Baltimore Ave, Philadelphia, PA`. Zoom to roughly 1:5,000 scale so block faces are visible.
5. Click **Save**. Title: `Lab02b_WalkingAudit_Map` · Tags: `walking audit, Lab02b`.

*[Figure: Map Viewer showing West Philadelphia aerial imagery at block scale, with the three sublayers listed in the Contents pane]*

---

## Part 5: Digitize the study-area polygon

1. Click **Edit** (pencil icon, left toolbar) → **Create features** → **study_area**.
2. Set the polygon fill to 50% transparency via **Styles** (right toolbar) so imagery shows through.
3. Click each street corner around the four-block boundary (four to six vertices). **Double-click** to close.
4. Leave `area_sqft` blank in the pop-up form. Click **Submit**.

**To edit vertices:** click **Select** in the editing toolbar → click the polygon. Drag vertex handles to adjust corners. Hover over a boundary segment to get a midpoint handle; drag to add a vertex.

---

## Part 6: Digitize street-segment lines

1. In the create-features panel, click **street_segments**. Click at one end of a block face; click successive points along it; **double-click** to finish.
2. In the pop-up form: enter `seg_id`, select coded values for `lighting`, `sidewalk`, `crossing`, `traffic_calm`, add a `notes` description. Leave `length_ft` blank. **Submit**.
3. Digitize at least **four block faces**, one per side of the study area.

**To edit:** click **Select** → click a line → drag vertex handles. To move the entire feature, drag the line itself.

> **Note:** Map Viewer does not display a snapping toolbar like ArcGIS Pro. For block-scale audit data, visual alignment over imagery is sufficient. Precise vertex snapping is available in **ArcGIS Field Maps** or ArcGIS Pro.

*[Figure: Map Viewer showing four polyline segments along block faces with the attribute pop-up form open for one segment]*

---

## Part 7: Digitize audit observation points

1. In the create-features panel, click **audit_sites**. Click a location on the map to place a point.
2. In the pop-up form: enter `site_id`; select coded values for `lighting`, `sidewalk`, `crossing`, `traffic_calm`; add `notes`. Leave `lat_dd` and `lon_dd` blank. **Submit**.
3. Digitize at least **six points** across the study area, representing a mix of conditions.

**To move a point:** click **Select** → click the point → drag to the correct location.

> **Warning:** If you place a point in the wrong location and the pop-up form appears, click **Discard** (or **Cancel**) before the feature is committed. Press **Escape** before the form appears to abort placement entirely.

**Viewing the table:** click **Tables** (left toolbar) → select `audit_sites`. All features and attributes appear. Click a row to highlight the feature on the map.

4. **Save** the web map.

---

## Part 8: Calculate geometry with Arcade

ArcGIS Online computes geometry **geodesically** — distances and areas are accurate regardless of the display projection (Web Mercator). You do not need to set a projected coordinate system before calculating, unlike the ArcGIS Pro workflow where choosing the correct CRS before running **Calculate Geometry** is critical.

Calculate each field from the hosted layer's item page (Content → `Lab02b_WalkingAudit` → click the sublayer → **Fields** tab → click the field → **Calculate values**).

| Sublayer | Field | Arcade expression |
|---|---|---|
| `street_segments` | `length_ft` | `Length($feature, 'feet')` |
| `study_area` | `area_sqft` | `Area($feature, 'square-feet')` |
| `audit_sites` | `lat_dd` | `Project(Geometry($feature), SpatialReference(4326)).y` |
| `audit_sites` | `lon_dd` | `Project(Geometry($feature), SpatialReference(4326)).x` |

Click **Run** after entering each expression. Return to the web map and open the **Tables** view to confirm the fields are populated.

> **Note:** If `lat_dd` returns a very large number (e.g., 4,400,000), the coordinates are still in Web Mercator meters, not decimal degrees. Make sure you are using the `Project(…, SpatialReference(4326))` form above.

---

## Part 9: Share the web map

ArcGIS Online has no print layout equivalent to ArcGIS Pro's **Layout** view. The deliverable is a shared web map link plus a screen capture.

1. Click **Save**, then **Share map** (top toolbar).
2. Under **Who can access this map**, select **Your organization**.
3. Click **Copy link**. Paste this URL into your submission.
4. Zoom to show all three layers. Take a browser screenshot.

> **Note:** For a polished map product, open the item page and click **Create app** → **Instant Apps** (Basic Viewer template) to add a title, legend, and zoom controls. Full cartographic layouts with north arrow and scalebar are best produced in **ArcGIS StoryMaps** or exported from ArcGIS Pro.

---

## ArcGIS Field Maps — the field-collection path

The hosted feature layer you built is the same structure used for real field data collection. **ArcGIS Field Maps** (iOS / Android) connects directly to any hosted layer in your ArcGIS org. A field auditor signs in, opens the map, taps **Collect here**, fills the coded-value form, and taps **Submit** — the feature is written to the hosted layer and immediately visible to all team members. The field names, types, and coded-value lists you defined in Part 3 determine exactly what field workers see on their mobile forms. This removes the paper form and subsequent manual digitizing step, and GPS location replaces on-screen clicking.

---

## Deliverables

1. **Web map URL** — the shared link to `Lab02b_WalkingAudit_Map`. Confirm your instructor can view it before submitting.
2. **Screenshot** — all three layers over imagery with the `audit_sites` table open showing at least six rows.
3. **`codebook_notes.txt`** — two-sentence reflection from Part 1, noting that the initial draft was AI-generated.
4. **Short interpretation (100 words):** Which one block face or intersection appears most hazardous, and what two coded conditions support that judgment?

---

## Using AI in this lab

**Permitted use (Part 1):** Draft an initial audit codebook with an AI assistant based on a 3-sentence study description. Use the result as a scaffold you then evaluate and refine.

**Guardrails that always apply:**

- AI helps with **scaffolding** (draft protocols, Arcade syntax), **explaining** (GIS concepts, error messages), and **critiquing** (field schema review). It does not make analytic decisions or interpret specific locations.
- Never paste PHI or identifiable addresses into a public AI tool. This lab uses illustrative records, but build the habit now.
- Label all AI-assisted output. Note in `codebook_notes.txt` that the initial draft was AI-generated and that you revised it.

---

## ArcGIS Pro equivalent

The matching desktop lab is **Lab 2b (ArcGIS Pro): Digitizing Built-Environment Walking-Audit Data**.

| ArcGIS Online | ArcGIS Pro equivalent |
|---|---|
| Content > New item > Feature layer | Catalog pane → New Feature Class wizard |
| Fields tab > List of values | Feature Class wizard Fields page + Domains manager |
| Create features sketch tool | Edit tab → Create Features panel |
| Pop-up form for attribute entry | Editing pane / attribute table row |
| Select + drag vertex handle | Edit tab → Modify Features → Vertices tool |
| Fields > Calculate values (Arcade) | Right-click field header → Calculate Geometry |
| Share map + browser screenshot | Insert tab → New Layout → Export Layout |
| ArcGIS Field Maps (mobile) | No desktop equivalent |

> **Note:** The most important workflow difference is projection. ArcGIS Online computes geometry geodesically and does not require you to set a projected CRS. In ArcGIS Pro, setting the map to NAD83 PA State Plane South (EPSG:2272) before digitizing is necessary for accurate length and area values in US feet.

---

## Check your understanding

1. You defined coded-value lists for four fields. What data quality problem do these lists prevent that a free-text field would not?
2. ArcGIS Online displays your map in Web Mercator (EPSG:3857). Does this make your Arcade-calculated segment lengths inaccurate? Explain why or why not.
3. A field auditor collects a GPS point in ArcGIS Field Maps. Where does that feature appear, and how quickly can a colleague in the office see it?
4. You run `Length($feature, 'feet')` on existing segments, then digitize two more. What must you do to populate `length_ft` for the new features?
5. Your instructor asks you to share the web map publicly (anyone with the link). What privacy consideration should you think about before changing the sharing setting?

## Public health question

Philadelphia's park system is a critical public health resource, but proximity depends on how
distance is measured. This lab asks: what share of Philadelphia children live within a 0.5-mile
walk of a public park, and how does that share vary by census-tract poverty quintile? ArcGIS Online
handles the distance calculation correctly by default — but understanding how and why lets you trust
your results and explain them to colleagues.

---

## What you will learn

- Understand why ArcGIS Online displays all layers in Web Mercator and what that means for analysis
- Recognize that Analysis tools compute **geodesic** distances — correct regardless of display projection
- Inspect a layer's source coordinate reference system (CRS) in item details
- Run **Create Buffers** at 0.5 miles and get a valid result without reprojecting data
- Use **Summarize Within** to aggregate child population inside park buffers by census tract
- Add a calculated field with **Calculate** (Arcade) to derive percent of children with park access
- Classify tracts by poverty quintile and apply a choropleth style
- Build and export a bar **Chart** of access by poverty quintile

---

## What you will produce

1. A **saved and shared web map** showing Philadelphia census tracts symbolized by park access rate,
   with 0.5-mile geodesic park buffers overlaid. Submit the item URL.
2. An **exported bar chart image** of the percentage of children with park access for each poverty
   quintile (1 = lowest poverty, 5 = highest).
3. A **completed access table** (the template in Part 4) with five quintile values filled in.
4. A **100-word written interpretation** identifying which quintile has the highest and lowest access
   rates, what the pattern suggests about park equity in Philadelphia, and why geodesic analysis
   matters here. Label any AI-assisted portions.

---

## Data

Your instructor will share the items below to your ArcGIS organizational account before the lab.

| Layer name | Description | Source |
|---|---|---|
| `Philadelphia_Parks` | Parks and recreation properties (polygons) | Instructor-shared item (OpenDataPhilly) |
| `Philadelphia_Tracts_ACS` | Census tracts with `CHILD_POP`, `POV_RATE`, and `POV_QUINTILE` | Instructor-shared item (NHGIS / ACS 5-year) |

> **Note:** `CHILD_POP` is total population under age 18 (ACS B09001). `POV_QUINTILE` is pre-coded
> (1 = lowest poverty, 5 = highest). Confirm exact field names in the item's **Fields** tab before
> beginning Part 3.

---

## Before you begin

**Account:** Sign in at **arcgis.com** using the course organizational account. A free public
ArcGIS account cannot run the Analysis tools used here.

> **Warning:** Running **Create Buffers** and **Summarize Within** consumes **ArcGIS credits**.
> Credit use for city-scale analyses is modest, but confirm with your instructor that credits are
> available and that your account has **Publisher** or **Analysis** privileges before running any tool.

**Prerequisites:** Lab 1 (adding data, basic navigation). No prior CRS knowledge needed.

**Time estimate:** 2 to 2.5 hours.

---

## Part 1: Open Map Viewer and add the data

1. Sign in at **arcgis.com**. From the app launcher (grid icon, top right), choose **Map Viewer**.
2. Click **Save**. Enter a title (`Lab02a_ParkAccess_[YourName]`), tags, and a summary. Click **Save**.
3. In the **Contents** toolbar (left, dark background), click **Layers** → **Add** → **Browse
   layers**. Switch to the **My Organization** tab.
4. Search for `Philadelphia_Parks`; click **Add**. Repeat for `Philadelphia_Tracts_ACS`.
5. Click **Save**.

*[Figure: Map Viewer showing Philadelphia parks and census tract boundaries over the default basemap]*

---

## Part 2: Understand how ArcGIS Online handles coordinate systems

### Inspect the source CRS

1. In the **Layers** pane, click the three-dot menu (`...`) next to `Philadelphia_Parks` and choose
   **View item details**. On the item page, scroll to the **Information** section and find
   **Coordinate System** — note whether it reads WGS 1984 (EPSG:4326) or Web Mercator (EPSG:3857).
2. Return to the map tab and repeat for `Philadelphia_Tracts_ACS`.

*[Figure: Item details page showing the Coordinate System field for Philadelphia_Parks]*

> **Note:** ArcGIS Online **displays** all layers in **Web Mercator (EPSG:3857)** regardless of
> their stored CRS. Layers with different source projections still line up on the map automatically.
> More importantly, Analysis tools do not use the display projection — they compute distances and
> areas **geodesically**, directly on the WGS84 ellipsoid. A buffer set to "0.5 miles" in
> **Create Buffers** is a true 0.5-mile radius no matter what CRS the source data use. You do not
> need to reproject data before running analysis in ArcGIS Online.
>
> **Why this is a desktop problem:** In ArcGIS Pro or QGIS, the default buffer measures in the
> layer's stored coordinate units. WGS84 data passed to a planar buffer produces a result in
> degrees — roughly 35 miles per unit at Philadelphia's latitude, not half a mile. The Pro version
> of this lab demonstrates that error deliberately. ArcGIS Online sidesteps it through geodesic
> computation, but knowing the risk makes you a more careful analyst in any platform.

3. Zoom to the `Philadelphia_Parks` layer (three-dot menu → **Zoom to layer**) and confirm the
   parks appear at city scale.

---

## Part 3: Create 0.5-mile geodesic buffers around parks

> **Warning:** **Create Buffers** consumes credits. Click **Estimate Credits** before running.

1. Click **Analysis** in the top toolbar to open the Analysis pane.
2. Search for **Create Buffers** and open the tool. Set:

| Parameter | Value |
|---|---|
| **Input layer** | `Philadelphia_Parks` |
| **Distance** | `0.5` |
| **Distance unit** | **Miles** |
| **Buffer overlap** | **Dissolve** |
| **Output layer name** | `Parks_Buffer_HalfMile` |

3. Click **Estimate Credits**, then **Run**. The buffer layer is added to the map when complete.

*[Figure: Map showing dissolved 0.5-mile park buffers covering portions of Philadelphia]*

4. Visually check the result. The rings should extend a modest distance from park edges, not cover
   the entire city. If the buffer looks wrong, check your distance value and units before proceeding.

---

## Part 4: Summarize child population within the buffer by tract

### Run Summarize Within

> **Warning:** **Summarize Within** also consumes credits. Estimate before running.

1. In the **Analysis** pane, search for **Summarize Within** and open the tool. Set:

| Parameter | Value |
|---|---|
| **Summary area** | `Philadelphia_Tracts_ACS` |
| **Layer to summarize** | `Parks_Buffer_HalfMile` |
| **Output layer name** | `Tracts_BufferOverlap` |

2. Click **Estimate Credits**, then **Run**. The output contains one record per tract, including the
   area of the buffer that overlaps each tract.

> **Note:** For tracts partially inside the buffer, **Summarize Within** uses areal interpolation:
> it apportions values by the share of tract area inside the buffer, assuming uniform population
> distribution within each tract. This is imperfect but standard practice at this scale.

### Calculate children with access per tract

3. Open the **Table** for `Tracts_BufferOverlap` (click the layer, then **Table** in the left
   toolbar). Note the field recording buffer overlap area (named something like `sum_Area_SquareMiles`)
   and confirm `CHILD_POP` carried over from the tract layer.

4. Click **Fields** (right toolbar) → **Add field**. Name it `PCTBUF` (Double). Click **Calculate**
   and enter the Arcade expression (adjust field names to match what you see in the table):

   `($feature.sum_Area_SquareMiles / AreaGeodetic($feature, 'squaremiles')) * 100`

   This gives the percentage of each tract's area inside the buffer.

5. Add a second field `CHILD_ACCESS` (Double). **Calculate**:

   `$feature.CHILD_POP * ($feature.PCTBUF / 100)`

   This estimates children in each tract who fall within the park buffer zone.

6. Click **Save** (map).

### Summarize by poverty quintile

7. Use **Filter** (right toolbar) to select `POV_QUINTILE = 1`. Read the sums of `CHILD_ACCESS`
   and `CHILD_POP` from the table status bar. Clear the filter, repeat for quintiles 2–5, and
   record below:

| Poverty quintile | Total children | Children with park access | % with park access |
|---|---|---|---|
| 1 (lowest poverty) | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 (highest poverty) | | | |

Divide `CHILD_ACCESS` by `CHILD_POP` and multiply by 100 per quintile.

---

## Part 5: Style the map and build a chart

### Classify tracts by park access rate

1. In the **Layers** pane, select `Tracts_BufferOverlap`. Open **Styles** (right toolbar).
2. Under **Choose attributes**, select `PCTBUF`. Choose the drawing style **Counts and Amounts
   (Color)**. Click **Style options**.
3. Set **Classification type** to **Quantile**, **Classes** to **5**. Choose a sequential color
   ramp (e.g., YlGn). Click **Done**.
4. Turn on `Parks_Buffer_HalfMile` and style it as a semi-transparent outline so the choropleth
   shows through. Click **Save**.

*[Figure: Choropleth map of Philadelphia tracts by percent area inside park buffers, with buffer outlines overlaid]*

### Build a bar chart

5. With `Tracts_BufferOverlap` selected, click **Charts** (left toolbar, bar-chart icon) →
   **Add chart** → **Bar chart**.
6. Set **Category (X axis)** to `POV_QUINTILE`, **Value (Y axis)** to `CHILD_ACCESS` (Sum).
   Title the chart "Children with Park Access by Poverty Quintile."
7. Click the **Export** icon to save the chart as a PNG image.

*[Figure: Bar chart with poverty quintile on the X axis and sum of estimated children with park access on the Y axis]*

---

## Deliverables

1. **Web map link:** Share your map with your organization and submit the item URL.
2. **Exported chart image:** The bar chart PNG from Part 5.
3. **Completed access table:** All five quintile rows filled in.
4. **100-word written interpretation:** Which quintile has the highest and lowest access rate?
   What does this suggest about park equity? Why does geodesic analysis matter for this result?
   Label any AI-assisted portions.

---

## Using AI in this lab

**Bounded AI task — interrogating Web Mercator and geodesic analysis:**

1. In ArcGIS Online, open the item details page for `Philadelphia_Parks`. Copy the **Coordinate
   System** text (e.g., "GCS WGS 1984" or "Web Mercator Auxiliary Sphere").
2. Paste it into **Claude** (claude.ai) with this prompt: "This is the coordinate system for a
   dataset in ArcGIS Online: [paste text]. Is it geographic or projected? When ArcGIS Online runs
   a buffer using geodesic computation, does the source CRS matter? Explain in plain language why
   geodesic analysis gives correct distances even when the display projection is Web Mercator."
3. Read the response critically. Does the AI correctly distinguish display projection from analysis
   method? Compare its explanation to what you observed in Part 2.
4. Optional: ask "If I ran a 0.5-mile buffer in ArcGIS Pro on WGS84 data using a planar buffer,
   what would go wrong?" Compare the response to the Part 2 warning.

**Guardrails:**

- AI helps with **explaining** (what is Web Mercator? why does geodesic matter?) and **critiquing**
  (is my interpretation of the quintile chart reasonable?). It does not make analytic decisions.
- AI does **not** select the right tool or interpret equity findings — those require your judgment.
- **Never paste patient data, addresses, or identifiable health information into a public AI tool.**
  The item detail text you copy here contains no personal information.
- Label any AI-assisted text in your submitted deliverables.

---

## ArcGIS Pro equivalent

The desktop version of this lab (Lab 2a, ArcGIS Pro) covers the same question using a
reproject-first approach. The table below maps the key steps.

| ArcGIS Online step | ArcGIS Pro equivalent |
|---|---|
| Inspect CRS in item details page | Layer Properties → Source → Spatial Reference |
| No reprojection needed; geodesic analysis | **Project** tool → EPSG:2272 (PA State Plane South, feet) |
| **Create Buffers** (0.5 miles, geodesic) | **Buffer** tool — input in feet (2,640 ft), planar mode |
| **Summarize Within** for area overlap by tract | **Intersect** + **Calculate Geometry** + **Field Calculator** |
| **Filter** by quintile + table | **Summary Statistics** with case field `POV_QUINTILE` |
| Charts panel + Share web map | Chart Properties + Export layout as PDF |

The core difference: ArcGIS Online's geodesic engine removes the reproject-before-buffer
requirement. The Pro lab demonstrates the degree-unit buffer error deliberately so students
recognize it in desktop workflows. Both platforms produce the same correct result when used properly.

---

## Check your understanding

1. ArcGIS Online displays all layers in Web Mercator, yet the buffer you created is correctly 0.5
   miles. How is that possible? What type of computation does ArcGIS Online use for distance analysis?

2. A colleague runs the same park buffer in ArcGIS Pro using data stored in WGS84 (EPSG:4326) and
   sets the buffer distance to 0.5, accepting the default units. What is likely wrong with her
   result, and how would you fix it?

3. **Summarize Within** uses areal interpolation when tracts partially overlap the buffer. What
   assumption does this make about population distribution? When might the assumption be most
   misleading?

4. Your bar chart shows that poverty quintile 5 tracts have lower estimated park access than quintile
   1 tracts. List two alternative explanations for this pattern — other than parks being located in
   wealthier areas — and describe what additional data you would need to evaluate each.

5. ArcGIS Online requires credits to run Analysis tools; ArcGIS Pro's geoprocessing tools do not.
   What does this trade-off suggest about when you might choose one platform over the other for
   routine public health GIS work?

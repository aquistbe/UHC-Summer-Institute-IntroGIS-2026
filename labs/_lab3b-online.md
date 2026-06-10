## Public health question

Philadelphia hosts dozens of facilities that report toxic chemical releases to the EPA's Toxic Release Inventory (TRI). Residents living near these facilities may face elevated exposure risk — but who lives nearby, and are those populations disproportionately composed of people of color? This lab uses buffer zones and overlay to estimate the population within 0–500 m, 500 m–1 km, and 1–2 km of TRI facilities and compares the racial/ethnic composition of that population to the rest of the city.

---

## What you will learn

- Add point data from a shared ArcGIS Online item and demographic data from the Living Atlas or via Enrich Layer
- Create multi-distance buffer rings using **Create Buffers** (geodesic — no reprojection needed)
- Use **Summarize Within** to apportion block-group population into each ring using areal apportionment
- Use **Enrich Layer** as an alternative workflow to pull race/ethnicity data directly from ArcGIS demographics
- Apply **Filter** and group **Statistics** to compare demographic composition inside vs. outside 1 km
- Style ring layers with transparency, build a legend, and share a web map
- Understand why ArcGIS Online does not require a projected CRS for distance operations, and what that means for reproducibility

---

## What you will produce

1. A saved and shared **web map** showing three concentric buffer rings (0–500 m, 500 m–1 km, 1–2 km) around Philadelphia TRI facilities, styled by ring distance, overlaid on block groups symbolized by percent non-White population.
2. A completed **demographic comparison table** (inside vs. outside 1 km) built from Summarize Within output and recorded in a Word document or course template.
3. A **short written interpretation** (approximately 100 words) describing what the table shows and two analytic limitations.
4. An **AI-corrected Methods paragraph** (see "Using AI in this lab").

> **Note:** ArcGIS Online does not have a print layout like ArcGIS Pro. You will share the web map directly. To produce a polished visual for a report, see the note in the Deliverables section on using ArcGIS Instant Apps or capturing a screenshot.

---

## Data

| Layer | Description | Source / How to access |
|---|---|---|
| `PHL_TRI_Facilities` | EPA Toxic Release Inventory point locations, Philadelphia subset | Instructor-shared ArcGIS Online item — find it under **My Organization** |
| ACS Block Groups — Philadelphia | ACS 2019–2023 5-year block groups with race/ethnicity fields | Living Atlas: search *"USA Census Block Groups"* or instructor-shared item under **My Organization** |
| ArcGIS Demographics (via Enrich) | Race/ethnicity variables from the ArcGIS demographic data package | Available through **Enrich Layer** when signed in to an org account with Enrich privileges |

---

## Before you begin

**Account:** Sign in to **arcgis.com** using your ArcGIS organizational account provided by the course. A free public ArcGIS account can view web maps but cannot run the analysis tools used in this lab.

> **Warning:** Running **Create Buffers**, **Summarize Within**, and **Enrich Layer** consumes **ArcGIS credits** from your organization. These tools also require **publisher or analysis privileges**. Use the course org account — not a personal free account — and do not run analysis tools more times than necessary. If you are unsure of your credit balance, ask the instructor before proceeding.

**Estimated time:** 2.5–3 hours.

**Prerequisites:** Completion of Labs 1, 2a, and 3a. You should know how to open Map Viewer, add layers, and apply styles.

**Open Map Viewer:**

1. Go to [arcgis.com](https://www.arcgis.com) and sign in with your org account.
2. Click the app launcher (the grid icon, top-right) and choose **Map Viewer**.
3. A new blank map opens. Click **Save and open** → **Save map**. Give it the title `Lab03b_TRI_Buffer_[YourInitials]`, add tags (`Lab03b`, `TRI`, `Philadelphia`), and click **Save**.

---

## Part 1: Add data

1. In the **Contents** toolbar (left side, dark panel), click **Layers**, then click **Add** (the + icon).

2. Choose **Browse layers**. In the search bar at the top of the panel, switch the scope from **My Content** to **My Organization**. Search for `PHL_TRI_Facilities`. When the layer appears, click **Add**. The TRI facility points appear on the map.

3. Click **Add** again → **Browse layers** → **My Organization**. Search for `PHL_BlockGroups_ACS`. Add that layer.

   *[Figure: Map Viewer showing TRI facility points and Philadelphia block group polygons added to the Contents panel]*

4. Zoom to the data: click the **Home** extent button, or right-click the block groups layer in the **Layers** panel and choose **Zoom to layer**.

5. In the **Layers** panel, click the block groups layer to select it. Open its **Table** (the table icon in the Contents toolbar). Identify these fields — you will use them in Parts 3 and 5:

   | Field | Contents |
   |---|---|
   | `GEOID` | Block group FIPS code (unique identifier) |
   | `TOTPOP` | Total population (ACS B03002_001E) |
   | `NH_WHITE` | Non-Hispanic White alone (B03002_003E) |
   | `NH_BLACK` | Non-Hispanic Black or African American alone (B03002_004E) |
   | `HISPANIC` | Hispanic or Latino of any race (B03002_012E) |
   | `SHAPE_AREA` | Block group area (in the layer's native units — square meters if projected) |

   Close the Table.

> **Note:** ArcGIS Online displays all layers in Web Mercator (EPSG:3857) regardless of the source CRS. You do **not** need to reproject layers before buffering. The **Create Buffers** tool computes geodesic distances — measured correctly on the Earth's surface — so a 1-km buffer is truly 1 km no matter what projection the display uses. This is the key difference from ArcGIS Pro, where buffering a layer stored in a geographic CRS (degrees) produces incorrect distances.

---

## Part 2: Create multi-distance buffer rings

> **Public health note:** Proximity to a TRI facility is a crude exposure proxy. Residents closer to a facility may face higher potential exposure, but actual risk depends on wind patterns, chemical properties, facility operating schedules, and individual behavior. This analysis screens for potential environmental inequity — it does not measure dose or confirm exposure.

> **Warning:** **Create Buffers** consumes credits. Confirm your parameters are correct before clicking **Run**.

1. In the **Contents** toolbar, click **Analysis** (the beaker icon). The Analysis panel opens.

2. In the search box, type `Create Buffers` and click the tool.

3. Set parameters:

   | Parameter | Value |
   |---|---|
   | Input layer | `PHL_TRI_Facilities` |
   | Distance | Select **Multiple distances**: enter `500`, `1000`, `2000` |
   | Distance unit | Meters |
   | Buffer type | Rings (not disks — this creates donuts so each ring covers only its distance band) |
   | Dissolve | Yes — dissolve all buffers for each distance into a single ring |
   | Output name | `TRI_Rings_3` |

4. Click **Run**. Three ring polygons appear: 0–500 m, 500–1000 m, and 1000–2000 m.

   *[Figure: Three concentric buffer rings over Philadelphia, one ring per distance band]*

5. Open the **Table** for `TRI_Rings_3`. You should see three rows, one per ring, with a `RingDistance` or `distance` field. If you see more rows, the dissolve did not apply — re-run with **Dissolve** set to Yes.

> **Note:** Unlike ArcGIS Pro's **Multiple Ring Buffer** tool (which defaults to nested disks), the ArcGIS Online **Create Buffers** tool with "Rings" selected creates true donut polygons — each ring covers only the band between its inner and outer distance. This means the ring polygons do not overlap, which simplifies the Summarize Within step.

---

## Part 3: Summarize population within each ring (areal apportionment)

**Summarize Within** overlays the ring polygons with the block group layer and apportions numeric fields by area. This is the ArcGIS Online equivalent of running Intersect and then multiplying by an area fraction in ArcGIS Pro.

> **Warning:** **Summarize Within** consumes credits. The tool will apportion continuous fields proportionally by the area of each block group that falls within the ring. This is the areal-apportionment assumption — read the Public health note below before running.

> **Public health note:** **Areal apportionment** assumes that population is distributed uniformly within each block group. If a block group overlaps the 1-km ring boundary, the tool assigns population proportional to the share of the block group's area that falls inside the ring. This assumption is almost never literally true: a block group split by a river or park may have most of its residents on one side. This is related to the **Modifiable Areal Unit Problem (MAUP)**: results change depending on the size and shape of the zones used. Use the apportioned totals as estimates, not census counts, and treat any comparison at the ring boundary with caution.

1. In the **Analysis** panel, search for `Summarize Within` and open the tool.

2. Set parameters:

   | Parameter | Value |
   |---|---|
   | Layer to summarize within | `TRI_Rings_3` |
   | Summary layer | `PHL_BlockGroups_ACS` |
   | Statistics to calculate | Sum of `TOTPOP`; Sum of `NH_WHITE`; Sum of `NH_BLACK`; Sum of `HISPANIC` |
   | Add minority count | No (use the explicit field sums above) |
   | Group by field | (leave blank) |
   | Output name | `Rings_SumWithin` |

3. Click **Run**. The output is a copy of the ring polygons with new fields containing the apportioned population sums for each ring.

   *[Figure: Attribute table of Rings_SumWithin showing sum fields for TOTPOP, NH_WHITE, NH_BLACK, and HISPANIC for each of the three rings]*

4. Open the **Table** for `Rings_SumWithin`. Record the sum values for each ring. You will use the 1-km ring row in Part 5.

---

## Part 4 (Alternative): Enrich Layer for the 1-km ring

**Enrich Layer** is a faster alternative when you only need the 1-km ring and do not have a block-group layer with the demographic fields you need. It pulls race/ethnicity data directly from Esri's demographic data package.

> **Warning:** **Enrich Layer** charges per enriched polygon and is typically more credit-intensive than Summarize Within. If you completed Part 3, you may skip this part. Ask the instructor before running if unsure about your credit balance.

1. Isolate the 1-km ring. Click `TRI_Rings_3` in the **Layers** panel. In the **Settings** toolbar (right), click **Filter**. Set the expression: `RingDistance` **is** `1000`. Click **Save**.

2. In the **Analysis** panel, search for `Enrich Layer` and open the tool.

3. Set parameters:

   | Parameter | Value |
   |---|---|
   | Input layer | `TRI_Rings_3` (filtered to 1 km) |
   | Variables to add | Expand **Race and Ethnicity** → select Non-Hispanic White, Non-Hispanic Black, Hispanic totals |
   | Output name | `Ring_1km_Enriched` |

4. Click **Run**. The output polygon gains demographic fields. Results may differ slightly from Part 3 because Enrich uses Esri's block-level dasymetric model rather than simple areal apportionment.

5. Remove the filter: click the layer → **Filter** → **Remove filter**.

> **Note:** Enrich Layer's underlying model is proprietary — you cannot inspect its assumptions. For a transparent, reproducible public health analysis, document which method you used and why, and prefer Summarize Within when you have the block-group data.

---

## Part 5: Compare demographics inside vs. outside 1 km

You have ring-level sums from Part 3. Subtract inside-1km values from citywide totals to get the outside group.

**Get citywide totals:**

1. Click `PHL_BlockGroups_ACS` in the **Layers** panel. Open its **Table**. Click **Statistics** at the bottom, select `TOTPOP`, and note the **Sum**. Repeat for `NH_WHITE`, `NH_BLACK`, and `HISPANIC`.

**Get inside-1km totals:**

2. From the `Rings_SumWithin` table (Part 3), record the sum fields for the 1-km ring row.

**Calculate outside totals and complete the comparison table:**

3. Outside = citywide total minus inside-1km total. Calculate percentages (group count ÷ zone total × 100). Fill in the table:

   | Group | Inside 1 km (n) | Inside 1 km (%) | Outside 1 km (n) | Outside 1 km (%) |
   |---|---|---|---|---|
   | Total population | | | | |
   | Non-Hispanic White | | | | |
   | Non-Hispanic Black | | | | |
   | Hispanic | | | | |
   | All other non-Hispanic | | | | |

> **Public health note:** If the table shows a higher share of non-White or Hispanic residents within 1 km of TRI facilities compared to the rest of the city, this is consistent with a large body of environmental justice research documenting disproportionate facility siting in communities of color. However, this proximity analysis cannot establish causation, cannot confirm actual chemical exposure, and does not account for historical zoning patterns, housing markets, or migration. Proximity is not exposure. The MAUP also applies: using census tracts instead of block groups would give different numbers, and the choice of the 1-km threshold is arbitrary.

---

## Part 6: Style the map

1. In the **Layers** panel, arrange layers in this order (top to bottom): `PHL_TRI_Facilities`, `TRI_Rings_3`, `PHL_BlockGroups_ACS`.

2. Click `TRI_Rings_3`. In the **Settings** toolbar (right), click **Styles**. Under **Pick a style**, choose **Types (Unique symbols)**. Set the field to `RingDistance` (or the ring distance field name in your layer). Assign:
   - 500 m ring: dark red, 40% transparency
   - 1000 m ring: orange, 40% transparency
   - 2000 m ring: yellow, 40% transparency

   Click **Done**.

3. Click `PHL_BlockGroups_ACS`. Open **Styles**. Under **Choose attributes**, select `NH_WHITE` ÷ `TOTPOP` — or, if the layer lacks a percent field, click **Fields** in the right toolbar, then **Add field** → name it `PCT_NONWHITE`, type Double, and use **Calculate** with the Arcade expression:

   ```
   (($feature["TOTPOP"] - $feature["NH_WHITE"]) / $feature["TOTPOP"]) * 100
   ```

   Then style on `PCT_NONWHITE` using **Counts and Amounts (Color)**, a light-to-dark sequential ramp (e.g., light yellow to dark purple), **Natural Breaks**, 5 classes.

4. Click `PHL_TRI_Facilities`. Open **Styles** → **Location (single symbol)**. Choose a distinct symbol (black circle, size 8 pt).

   *[Figure: Styled web map showing Philadelphia block groups shaded by percent non-White, three concentric TRI buffer rings, and TRI facility points]*

5. Click **Save** (top of the map).

---

## Deliverables

Submit the following to the course portal:

1. **Web map URL:** After saving, click **Share map** (the share icon, upper right). Share to your organization. Copy the link and paste it into your submission document.

   > **Note:** ArcGIS Online has no print layout like ArcGIS Pro. For a shareable static image, use your browser's screenshot tool and crop to the map area, or open the map in an **Instant App** (click **Create app** → **Instant Apps** → choose a simple viewer template) to add a title and legend for a more polished appearance.

2. **Comparison table:** The completed inside/outside 1 km demographic table from Part 5, in your Word document or course template.

3. **Written interpretation (approximately 100 words):** Describe what the comparison table shows. Address at least two of the following: (a) direction and magnitude of any racial/ethnic difference; (b) the areal-apportionment assumption and why it matters; (c) why proximity does not equal exposure; (d) one alternative analysis that would strengthen the inference.

4. **AI-corrected Methods paragraph** (see "Using AI in this lab" section): the AI draft with your tracked-changes corrections.

Label any AI-assisted output with the notation `[AI-assisted, reviewed by <your name>]`.

---

## Using AI in this lab

**Bounded task — draft a Methods paragraph, then fix it.**

After completing Part 5 and filling in your comparison table, paste the values into Claude (claude.ai) or another AI assistant with this prompt:

> "I ran a buffer-and-overlay analysis in ArcGIS Online to compare racial/ethnic composition inside versus outside 1 km of EPA Toxic Release Inventory facilities in Philadelphia. I used Create Buffers (1 km geodesic ring), Summarize Within to apportion block-group population by area, and calculated citywide totals by subtraction. Here are my summary counts: [paste your table]. Write a 150-word Methods paragraph suitable for a community health report, describing what I did and what the numbers show."

Read the AI's draft carefully. It will likely make one or more of the following errors:

- Describe areal apportionment as producing exact population counts rather than estimates
- Omit the MAUP limitation or treat the 1-km threshold as a scientifically established standard
- Imply that residents near TRI facilities are necessarily exposed to toxic chemicals
- Overstate the certainty of the demographic comparison

**Your task:** In your Word document, paste the AI draft, then use **Track Changes** to correct every error you can identify. Write a one-sentence note after each correction explaining why it was wrong.

**Guardrail:** AI helped you draft narrative text. It did not choose the buffer distances, select the datasets, decide which demographic groups to compare, or interpret any finding. Those decisions belong to you as the analyst. Never paste identifiable health data, patient addresses, or any protected health information into a public AI tool.

---

## ArcGIS Pro equivalent

The ArcGIS Online workflow maps closely to the Pro workflow, with two key differences: no reprojection step (Online computes geodesic distances automatically) and **Summarize Within** replaces the manual Intersect-plus-field-calculator chain.

| ArcGIS Online tool | ArcGIS Pro equivalent |
|---|---|
| **Create Buffers** (multiple distances, geodesic) | **Multiple Ring Buffer** (requires projected CRS in meters) |
| **Summarize Within** (areal apportionment built in) | **Intersect** + **Calculate Geometry** + **Field Calculator** |
| **Enrich Layer** | No direct equivalent — requires joining external ACS tables |
| **Filter** (by ring distance) | **Select By Attributes** |
| **Statistics** on table | **Summary Statistics** geoprocessing tool |
| Share web map + Instant App | **Export Layout** to PDF |

The matching desktop lab (Lab 3b: Buffer and Overlay for Environmental Justice Screening — ArcGIS Pro) covers manual CRS verification, step-by-step areal weighting, the Erase tool for the outside-1km set, and a PDF print layout.

---

## Check your understanding

1. In ArcGIS Pro, running **Multiple Ring Buffer** on a layer stored in a geographic CRS (degrees) produces incorrect distances. Why does this problem not arise in ArcGIS Online when you use **Create Buffers**?

2. **Summarize Within** apportions population by area. A block group on the edge of the 1-km ring covers 0.8 km², but only 0.3 km² falls inside the ring. How does the tool estimate how many people from that block group live inside the ring? What assumption does this make, and when would it be most likely to mislead you?

3. A colleague suggests using **Enrich Layer** instead of **Summarize Within** because it is faster. What is one advantage and one disadvantage of Enrich Layer compared to Summarize Within for this analysis?

4. Your comparison table shows that 62% of residents within 1 km of TRI facilities are non-White, versus 41% citywide. A local advocacy group wants to publish this as evidence that "TRI facilities are causing health harm in communities of color." Identify two specific methodological problems with that claim as stated.

5. You ran **Create Buffers** with a 1-km ring around all TRI facilities and dissolved them into a single polygon. A different analyst buffers each facility individually without dissolving, then unions the results. Would the two approaches produce the same population estimate from **Summarize Within**? Explain why or why not.

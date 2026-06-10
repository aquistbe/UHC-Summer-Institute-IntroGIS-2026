## Public health question

Federally Qualified Health Centers (FQHCs) are a primary safety-net resource for uninsured and underinsured Philadelphians. Proximity on a map does not equal access. If residents cannot reach an FQHC within a reasonable travel time, that center does not serve them in practice. This lab asks: what share of West Philadelphia residents live more than a 15-minute drive from the nearest FQHC?

## What you will learn

- Add FQHC point data from a shared organizational item and ACS tract population from the Living Atlas
- Use **Create Drive-Time Areas** to generate 15-, 30-, and 45-minute catchment polygons
- Distinguish overlapping disks from dissolved rings and choose the right option for a coverage question
- Use **Find Nearest** to link residential points to their nearest FQHC with travel-time attributes
- Use **Summarize Within** to estimate covered vs. uncovered tract population
- Style catchment polygons and share a finished web map

## What you will produce

- A saved, shared ArcGIS Online web map with three drive-time rings (15/30/45 min) around Philadelphia FQHCs, symbolized with a clear legend
- A **Find Nearest** result layer showing which FQHC each West Philadelphia sample point is routed to, with travel time in the pop-up
- A **Summarize Within** output with estimated tract population inside and outside the 15-minute catchment, plus a calculated percent uncovered
- A 150-word written interpretation answering the public health question

## Data

| Layer | Description | Source |
|---|---|---|
| FQHC locations | Point locations of Philadelphia FQHCs | Instructor-shared item (`phl_FQHC_points`) — search **My Organization** |
| West Philadelphia sample points | ~50 residential address points in West Philadelphia | Instructor-shared item (`westphilly_sample_pts`) — search **My Organization** |
| ACS Tract Population | Census tract boundaries with ACS 5-year total population | **Living Atlas** — search `ACS Population Variables Tracts` |

> **Note:** The FQHC and sample-point layers are pre-loaded in your organizational account by your instructor. The ACS tract layer comes from the Living Atlas; no file download is needed.

---

## Before you begin

**Prerequisites:** Complete Labs 1–3. You should be able to add layers, apply a style, and run an Analysis tool.

**Time estimate:** 90–120 minutes.

**Accounts, credits, and privileges:** Running **Create Drive-Time Areas**, **Find Nearest**, and **Summarize Within** consumes **credits** and requires **publisher or analysis privileges**. A free ArcGIS public account cannot run these tools. Use the **ArcGIS organizational account** your course provides. Do not click **Run** more than once per step.

**Setup:** Go to **arcgis.com** and sign in. From the app launcher (grid icon, upper right), choose **Map Viewer**. Click **New Map**. Immediately **Save** with a title (`Lab04a_YourName`), tags, and a one-sentence summary.

---

## Part 1: Add data to the map

1. In the **Contents** toolbar (left, dark), click **Layers** → **Add** → **Browse layers**.
2. Switch scope to **My Organization**. Search for `phl_FQHC_points`. Click **+** to add it.
3. Repeat to add `westphilly_sample_pts`.
4. Switch scope to **Living Atlas**. Search for `ACS Population Variables Tracts`. Add the Esri layer containing total population at the tract level.
5. In **Styles** (right toolbar), set the ACS tract layer to **Location (single symbol)** with a light-gray outline and no fill — it should serve as a reference boundary, not dominate the map.
6. **Save** the map.

*[Figure: Map Viewer showing FQHC points, West Philadelphia sample points, and gray tract outlines across Philadelphia]*

---

## Part 2: Create Drive-Time Areas

A drive-time catchment polygon shows every location reachable within a given time from a facility, following actual street geometry. This is more informative than a straight-line buffer.

> **Warning:** **Create Drive-Time Areas** is a routing tool that consumes credits. With a handful of FQHCs and three break values, the cost is small — but estimate credits before running and do not click **Run** more than once.

1. In the **Contents** toolbar, click **Analysis** (the hexagon icon).
2. Search for `Create Drive-Time Areas`. Open the tool.
3. Set parameters:

| Parameter | Value |
|---|---|
| Input layer | `phl_FQHC_points` |
| Travel mode | Driving Time |
| Break values | `15 30 45` |
| Break units | Minutes |
| Area type | **Rings** |
| Output name | `FQHC_DriveTime_Rings` |

> **Note — Rings vs. Disks:** **Disks** produce overlapping full polygons (0–15, 0–30, 0–45 min). **Rings** produce non-overlapping bands so each location appears in only one zone. Use **Rings** for this coverage question; use **Disks** if you need to count how many facilities overlap at a point.

4. Click **Estimate credits**, then click **Run**. Wait 1–3 minutes.

*[Figure: Three nested drive-time rings (15, 30, 45 min) around FQHC points, with tract outlines visible underneath]*

5. Select the `FQHC_DriveTime_Rings` layer. Open **Styles** → choose the break-value field (often `ToBreak`) → **Types (Unique symbols)**. Assign green / yellow / orange to the three rings at ~50% opacity. Edit **Legend** labels to read "0–15 min", "15–30 min", "30–45 min".

> **Public health note:** A straight-line buffer treats every direction as equally accessible. A network drive-time polygon follows real streets and respects barriers. West Philadelphia is bounded on the east by the Schuylkill River — notice whether the drive-time rings are constrained where a buffer would not be. More critically, drive-time overstates access for carless patients. Safety-net populations disproportionately rely on transit, walking, or paratransit; a 15-minute drive-time catchment likely covers a different population than a 15-minute transit catchment. Interpret these results as a first screen, not a complete access assessment.

---

## Part 3: Find Nearest FQHC from residential points

**Find Nearest** identifies the closest facility to each origin and returns a travel-time attribute — a route-level answer for each residential location.

> **Warning:** **Find Nearest** consumes credits proportional to the number of origin-destination pairs. With ~50 sample points and a handful of FQHCs, the cost is small. Do not run with a large origin set without first estimating credits.

1. In the **Analysis** pane, search for `Find Nearest`. Open the tool.
2. Set parameters:

| Parameter | Value |
|---|---|
| Input layer (origins) | `westphilly_sample_pts` |
| Near layer (destinations) | `phl_FQHC_points` |
| Measurement type | Driving time |
| Maximum number of nearest | `1` |
| Output name | `WestPhilly_NearestFQHC` |

3. Click **Estimate credits**, then click **Run**.
4. Open the output layer's **Table**. Find the travel-time field (often `Total_TravelTime` or `TravelTime_min`). Scan the values: are any sample points more than 15 minutes from their nearest FQHC?

*[Figure: Connector lines from West Philadelphia residential points to their nearest FQHC, labeled with travel times]*

> **Note:** **Find Nearest** returns connector lines (origin-to-destination) with network-computed travel times — not turn-by-turn route geometry. To visualize the actual routed path, use **Plan Routes** or **Connect Origins to Destinations** (both available under Analysis, both consume credits). For this lab, the travel-time attribute from **Find Nearest** is sufficient.

---

## Part 4: Estimate covered and uncovered population

**Summarize Within** sums tract population fields for tracts that fall inside the 15-minute ring.

> **Warning:** **Summarize Within** consumes credits. Set parameters carefully before running.

1. Apply a **Filter** to `FQHC_DriveTime_Rings` (right toolbar) so only the 15-minute ring is active: field `ToBreak` **is** `15`. Confirm the field name and value in the layer's Table before filtering.
2. In the **Analysis** pane, search for `Summarize Within`. Open the tool.
3. Set parameters:

| Parameter | Value |
|---|---|
| Area layer | `FQHC_DriveTime_Rings` (filtered to 15-min ring) |
| Input layer | ACS tract layer |
| Summary field | Total population field (check name in Table — often `B01003_001E` or `TotalPop`) |
| Summary statistic | Sum |
| Output name | `Pop_Within_15min` |

4. Click **Estimate credits**, then click **Run**.
5. Open the output Table. Record the **Sum** — this is the tract population covered within 15 minutes.
6. For the total study area population, open the ACS tract layer's Table and use column **Statistics** on the population field. Subtract covered from total to get uncovered. Calculate `uncovered / total × 100`. Record the percent uncovered.

> **Warning:** Assigning an entire tract as covered or uncovered based on its polygon's relationship to the ring misclassifies tracts that straddle the boundary. This is a known limitation of zone-based population estimation. Note it in your written interpretation.

> **Public health note:** This estimate assumes all residents drive and that travel time is the only barrier. In practice, cost, insurance status, clinic hours, language, and patient load all constrain effective access. The drive-time catchment is a useful first screen, not a complete access assessment.

---

## Deliverables

1. **Shared web map URL:** Save and share `Lab04a_YourName` with your organization. Submit the item URL. The map must show the three drive-time rings, the sample points, and the Find Nearest layer with a visible legend.

> **Note:** ArcGIS Online has no full print-layout tool equivalent to ArcGIS Pro's Layout view. For a static image, use a browser screenshot or the **Print** widget. For a polished deliverable, embed the web map in an **ArcGIS StoryMap** or **Instant App**.

2. **Population summary table:** Total Philadelphia tract population, covered within 15 min, not covered, and percent uncovered.
3. **150-word written interpretation:** Answer the public health question. Address: (a) the share of West Philadelphia residents appearing underserved; (b) at least one limitation of the drive-time method for a safety-net population; (c) one policy or programmatic implication.
4. **Label any AI-assisted output** with the prompt used and how you verified the result.

---

## Using AI in this lab

**Suggested use — planning the workflow:** Ask an AI assistant: *"I need to estimate the share of West Philadelphia residents who live more than 15 minutes by car from a Federally Qualified Health Center using ArcGIS Online. Translate that public health question into a step-by-step ArcGIS Online tool sequence, then verify each step against Esri's documentation."* Compare the response against this handout before proceeding.

**Suggested use — troubleshooting output:** If drive-time rings look wrong, ask: *"In ArcGIS Online Create Drive-Time Areas, why might a ring extend into an area with no road access?"* Use the explanation to guide troubleshooting.

**Guardrails — required:**

- AI helps with scaffolding (draft steps, expressions, text), explaining (concepts, errors), and critiquing (map design, interpretation). That is all.
- AI does not choose datasets, make analytic decisions, or interpret results for your specific study area.
- Never paste patient addresses, identifiable health records, or any PHI into a public AI tool.
- Label all AI-assisted output in your submission with the prompt used and how you verified the result.

---

## ArcGIS Pro equivalent

| ArcGIS Online tool | ArcGIS Pro equivalent |
|---|---|
| **Create Drive-Time Areas** | **Network Analyst → Service Area** (cutoffs 15/30/45; Output Geometry = Rings) |
| **Find Nearest** | **Network Analyst → Closest Facility** (Incidents = residential points; Facilities = FQHCs; Find = 1) |
| **Plan Routes / Connect Origins to Destinations** | **Network Analyst → Route** (add stops in sequence) |
| **Summarize Within** | **Select By Location** (tracts within 15-min polygon) + **Summary Statistics** (SUM of `TOT_POP`) |

The Pro version of this lab (Lab 4a: Network Analysis for Healthcare Access) covers all four tools with a local street network dataset. The key practical difference: ArcGIS Online uses Esri's hosted routing service (no local dataset required, but credits consumed); Pro can use a local network dataset or the same hosted service. ArcGIS Online's standard Analysis tools do not offer a transit travel mode — transit-based access analysis requires a third-party transit network such as OpenRouteService with GTFS data.

---

## Check your understanding

1. What is the difference between **Rings** and **Disks** in **Create Drive-Time Areas**? For a question about which residents fall outside the 15-minute catchment, which is more appropriate, and why?
2. Why does **Create Drive-Time Areas** consume credits while adding a Living Atlas layer does not?
3. A colleague proposes buffering the FQHC points by 1 mile as a proxy for 15-minute drive-time access. When would a buffer and a drive-time catchment give similar results? When would they diverge most — and why does that matter for West Philadelphia specifically?
4. The lab uses driving time as the travel mode. For a safety-net patient population, what mode would be more appropriate? What additional data would a transit-time analysis require, and why is that not straightforward in ArcGIS Online?
5. Your population estimate assigns each tract fully inside or outside the 15-minute ring. Name one alternative approach that would be more accurate, and describe the trade-off it involves.

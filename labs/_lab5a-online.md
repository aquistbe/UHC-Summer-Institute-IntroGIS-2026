## Public health question

Philadelphia has only a handful of EPA AQS PM2.5 monitors, and they are not evenly distributed across the city. From these sparse point measurements, can we estimate what summer PM2.5 concentrations look like across the city — and, just as importantly, where should we trust that estimate?

This lab uses the **Interpolate Points** analysis tool in ArcGIS Online to generate a continuous PM2.5 surface from monitor points. You will run the interpolation, review the cross-validation statistics it reports, rerun with different settings to compare results, and identify where uncertainty is highest because monitors are far away.

---

## What you will learn

- Add point and polygon layers to a web map from instructor-shared items
- Run **Interpolate Points** and interpret its parameters and cross-validation output
- Compare two runs with different optimization and class settings
- Style a continuous surface with a sequential color ramp
- Use the prediction error layer and a density surface to identify areas of low reliability
- Share a web map as a deliverable

---

## What you will produce

1. A saved, shared ArcGIS Online web map with your interpolated PM2.5 surface styled with a sequential color ramp
2. A screenshot or browser-print of the web map with legend visible
3. A two-run comparison table with cross-validation RMSE from each run
4. A 150-word written interpretation addressing method choice, surface differences, and spatial uncertainty

---

## Data

| Layer | Description | Source |
|---|---|---|
| `phl_air_monitors` | EPA AQS PM2.5 monitor points; field `PM25_MEAN` = summer mean (µg/m³) | Instructor-shared ArcGIS Online item |
| `phl_boundary` | Philadelphia city boundary polygon | Instructor-shared ArcGIS Online item |

> **Note:** ArcGIS Online displays layers in Web Mercator and reprojects on the fly. **Interpolate Points** computes distances geodesically, so no manual projection step is needed — a key difference from the ArcGIS Pro version of this lab, where a projected CRS is required.

---

## Before you begin

**Account required:** Sign in at [arcgis.com](https://arcgis.com) using the **ArcGIS organizational account** provided for this course. A free public account cannot run Analysis tools.

> **Warning:** **Interpolate Points** consumes **credits** and requires **publisher/analysis privileges**. Each run costs credits based on input points and output resolution. Use only the runs described here. Ask your instructor about credit limits before clicking **Run**.

**Prerequisites:** Adding layers, Contents and Settings toolbars, basic navigation (Labs 1–3).

**Estimated time:** 60–90 minutes.

---

## Part 1: Set Up the Web Map

1. Go to [arcgis.com](https://arcgis.com) and sign in to your organizational account.

2. From the app launcher (grid icon, top right) choose **Map Viewer**.

3. Click **Save and open** → **Save as**. Title the map `Lab05a_PM25_Interpolation_[YourName]`; add tags `PM2.5`, `interpolation`, `Philadelphia`, `Lab05a`; add a brief summary. Click **Save**.

4. In the **Contents** (left) toolbar, click **Layers** → **Add** → **Browse layers**. Change the dropdown to **My Organization**, search for `phl_air_monitors`, and click **Add**.

5. Repeat to add `phl_boundary`. In the Layers pane, drag `phl_boundary` below `phl_air_monitors`.

   *[Figure: Web map showing Philadelphia boundary and PM2.5 monitor points]*

6. Select `phl_air_monitors`, then click **Table** in the Contents toolbar. Locate `PM25_MEAN` and note the range of values (summer mean PM2.5 in µg/m³). Close the table.

   > **Public health note:** The EPA annual PM2.5 standard is 9 µg/m³ (revised 2024); the 24-hour standard is 35 µg/m³. Higher monitor values indicate greater respiratory and cardiovascular risk, especially for children, older adults, and people with pre-existing lung disease.

7. With `phl_air_monitors` selected, open the **Settings** (right) toolbar → **Styles**. Set the attribute to `PM25_MEAN` and choose **Counts and Amounts (Size)** so higher values appear as larger circles. Click **Done**.

   *[Figure: Monitor points symbolized by PM25_MEAN size]*

   > **Warning:** You have very few monitor points — deliberately. Interpolation produces a smooth surface regardless of sample size, but smoothness is not accuracy. Keep this in mind throughout the lab.

---

## Part 2: Run Interpolate Points — First Run

**Interpolate Points** uses Empirical Bayesian Kriging (EBK) automatically, fitting the spatial structure of the data and reporting cross-validation statistics.

> **Warning:** Each **Run** click consumes credits. Set all parameters before running. Do not run the tool more than the two times described here without instructor direction.

8. In the Contents toolbar, click **Analysis**. Search for `Interpolate Points` and open the tool.

9. Set the first-run parameters:

   | Parameter | Value |
   |---|---|
   | Input layer | `phl_air_monitors` |
   | Field to interpolate | `PM25_MEAN` |
   | Optimize for | **Speed** |
   | Number of classes | `5` |
   | Output name | `PM25_Speed5` |

10. Click **Estimate credits**, then **Run**.

11. When complete, the interpolated surface and a prediction error layer appear in the map. A **cross-validation summary** panel shows **Root Mean Square Error (RMSE)**.

    *[Figure: Interpolated PM2.5 surface with cross-validation summary panel]*

12. **Record the RMSE** in your comparison table (Part 4). Note any sample-size warnings.

    > **Note:** EBK averages predictions across many semivariogram models, which helps with small samples. Even so, fewer than a dozen monitors means the RMSE reflects genuine uncertainty about PM2.5 across the full city.

---

## Part 3: Rerun with Different Settings — Second Run

> **Warning:** This second run also consumes credits.

13. Open **Interpolate Points** again with these settings:

    | Parameter | Value |
    |---|---|
    | Input layer | `phl_air_monitors` |
    | Field to interpolate | `PM25_MEAN` |
    | Optimize for | **Accuracy** |
    | Number of classes | `9` |
    | Output name | `PM25_Accuracy9` |

14. Click **Estimate credits**, then **Run**.

15. Record the RMSE from the second run's cross-validation summary.

16. Toggle the two interpolated layers on and off in the Layers pane to compare surfaces visually. Note where they differ most.

    *[Figure: Toggling between Speed/5-class and Accuracy/9-class surfaces]*

---

## Part 4: Complete the Comparison Table

17. In your lab write-up, complete this table:

    | Run | Optimize for | Classes | RMSE | Notes |
    |---|---|---|---|---|
    | 1 — `PM25_Speed5` | Speed | 5 | | |
    | 2 — `PM25_Accuracy9` | Accuracy | 9 | | |

    Lower RMSE means lower average prediction error at leave-one-out monitor locations. More classes can reveal finer surface detail but do not by themselves improve prediction accuracy.

---

## Part 5: Style and Explore the Surface

18. In the Layers pane, select `PM25_Accuracy9` as your primary surface.

19. Open **Settings** → **Styles**. Confirm **Counts and Amounts (Color)** is selected. Click **Style options** and set:

    | Option | Value |
    |---|---|
    | Color ramp | Yellow to dark orange/red (higher = darker) |
    | Classification | Natural breaks |
    | Classes | 5 |

20. Click **Done**.

    *[Figure: Interpolated PM2.5 surface with yellow-to-red sequential ramp and Philadelphia boundary]*

21. In the Layers pane, find the prediction error layer created by **Interpolate Points** (named something like "standard error" or "uncertainty"). Toggle it on. Open its **Styles** and apply a sequential ramp where darker = higher standard error = less reliable prediction.

    *[Figure: Prediction standard error layer; darker areas indicate greater uncertainty]*

    > **Warning:** A smooth surface gives no visual cue about reliability. The standard error layer is essential context: a neighborhood may appear to have a precise PM2.5 value when no monitor is within several kilometers.

    > **Public health note:** A modeled exposure surface is not a measured exposure. Do not use it for regulatory decisions, health alerts, or fine-scale individual exposure estimates. Appropriate uses include hypothesis generation, identifying monitor gaps, and contextualizing community health data — always with explicit uncertainty statements.

---

## Part 6: Monitor Density as an Uncertainty Proxy

The standard error layer quantifies model uncertainty. A more intuitive complement is a map of where monitors are sparse.

22. In the Analysis pane, search for **Calculate Density** and open it.

    > **Note:** ArcGIS Online's standard Analysis panel does not include a standalone Euclidean-distance raster tool. **Calculate Density** produces a kernel density surface showing where monitors cluster; low-density areas are far from monitors and serve as a useful reliability proxy. If your account has **Raster Analysis** privileges, a **Euclidean Distance** raster function is available under **Raster Functions** — ask your instructor.

    > **Warning:** **Calculate Density** also consumes credits.

23. Set parameters:

    | Parameter | Value |
    |---|---|
    | Input layer | `phl_air_monitors` |
    | Field | (none) |
    | Output name | `Monitor_Density` |

24. Click **Run**. Style the result with a reversed ramp: dark where density is low (monitor-sparse), light where density is high. Toggle it alongside `PM25_Accuracy9` to see which neighborhoods have the weakest data support.

    *[Figure: Monitor density surface alongside interpolated PM2.5; dark areas have few nearby monitors]*

---

## Part 7: Share the Map

25. In the Layers pane, set the final visible layers: `phl_boundary` (outline, no fill), `PM25_Accuracy9`, and `phl_air_monitors`. Turn off the density and error layers for the submission view.

26. Click **Save and open** → **Save**.

27. Click **Share map**. Under **Who can view**, select **Your organization**. Click **Save**. Copy the web map URL — this is your primary deliverable link.

28. Capture the map as an image. Use your browser's **Print** → **Save as PDF** and name it `Lab05a_Map_[YourName].pdf`.

    *[Figure: Final web map: PM2.5 surface, legend, monitor points, city boundary]*

---

## Deliverables

1. **Web map URL** — shared to your organization. Confirm the link opens before submitting.
2. **Map image** (`Lab05a_Map_[YourName].pdf` or `.png`) — final view with legend visible.
3. **Comparison table** — two-run table from Part 4 with RMSE values filled in.
4. **150-word written interpretation** — which run had lower RMSE; what the surface differences reveal about sparse-data interpolation; where predictions are least reliable and why that matters for health equity analysis. Label with your name.

---

## Using AI in this lab

**Bounded AI task:** Ask an AI assistant (e.g., Claude): *"What is the main practical difference between Inverse Distance Weighting and Empirical Bayesian Kriging for estimating air pollution concentrations, and what are the trade-offs for a public health analyst working with sparse monitor data?"*

Compare the response to your course readings. In your write-up, note one point where the AI was clear and useful, and one point where the reading provided nuance the AI missed.

**Guardrails:**

- AI is appropriate for **scaffolding** (drafting a concept explanation), **explaining** (what RMSE means, why smoothness does not signal accuracy), and **critiquing** (reviewing your written interpretation).
- AI does **not** choose appropriate interpolation settings, assess surface accuracy, or interpret spatial health patterns.
- **Never paste identifiable data** — addresses, names, or patient-level information — into a public AI tool.
- **Label all AI-assisted output**: e.g., *"Drafted with AI assistance and revised by the student."*

---

## ArcGIS Pro equivalent

| ArcGIS Online | ArcGIS Pro |
|---|---|
| **Interpolate Points** (EBK, automatic) | Geostatistical Wizard → **Empirical Bayesian Kriging** or **IDW** |
| Cross-validation RMSE in results pane | Cross-validation window in Geostatistical Wizard (RMSE, standardized RMS) |
| **Calculate Density** as uncertainty proxy | **Euclidean Distance** (Spatial Analyst) + **Extract by Mask** |
| Styles → sequential ramp | Symbology pane → Classify → color scheme |
| Share map URL | Export Layout → PDF; publish as web layer |
| No projection step | Projected CRS required; mismatch causes distance errors |

The Pro lab (Lab 5a: ArcGIS Pro) runs IDW at three power settings and compares Ordinary Kriging alongside EBK for a fuller method comparison. The Online lab focuses on EBK parameter settings because **Interpolate Points** uses EBK exclusively. Running both labs deepens understanding of what EBK automates.

---

## Check your understanding

1. **Interpolate Points** uses EBK, not IDW. Name one way EBK is statistically richer than IDW — and one reason that advantage may shrink when you have very few monitors.

2. Your Accuracy run has a lower RMSE than the Speed run. Does that mean the Accuracy surface is definitely better for public health use? What else would you want to know?

3. A colleague says: "The map shows PM2.5 is uniformly low in Northeast Philadelphia." What is the flaw in that interpretation, and how do the standard error or density layers help you respond?

4. You have eight monitors in a city of 142 square miles. A community group asks for the PM2.5 concentration at a specific block with no nearby monitor. What cautions do you include?

5. ArcGIS Online skips the coordinate system step that ArcGIS Pro requires. Explain briefly why Pro needs a projected CRS for interpolation and what goes wrong without one.

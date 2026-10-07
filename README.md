# Cyclone Anomaly Detection

This project tackles a common industrial challenge: predicting and identifying faults in a preheater cyclone using raw, unlabeled sensor data. 

### The Problem & Data
We are working with 3.6 years of historical telemetry data from a cyclone separator. The dataset contains 5-minute interval readings for temperatures and draft pressures. 

The core problem is that we do not have labeled "faults" to train a standard machine learning model. Instead, we must separate normal operating behavior from abnormal patterns, taking into account that the plant's "normal" state slowly drifts over months due to wear and seasonal changes.

### What We Are Identifying
We want to pinpoint **abnormal operating periods**—sustained operational issues (lasting 12+ hours) such as blockages, draft instability, and over-temperatures—while deliberately ignoring short-term sensor noise and planned plant shutdowns.

---

### Data Cleaning & Methodology
We avoided complex black-box machine learning models. Instead, we used a transparent, physics-based statistical approach:
* **Cleaning**: Fixed misaligned date formats that scrambled the timeline. We explicitly removed communication errors (e.g., `Not Connect`, `I/O Timeout`) and fake 0°C or 1375°C readings. We **did not** interpolate missing gaps, as inventing data hides genuine downtime.
* **Feature Engineering**: Faults break the *relationship* between sensors. We tracked features like `dT` (inlet gas temp minus outlet gas temp) and pressure drops.
* **Methodology**: 
  * We computed **Robust Z-Scores** against a **45-day rolling median**. 
  * This measures how many "normal-sized wiggles" a sensor is away from its recent baseline. 
  * If a feature stayed severely out of bounds for over 12 continuous hours, it was flagged as an abnormal period.

---

### Results
Out of roughly 24,000 running hours, we successfully identified **22 abnormal periods** (about 6% of running time). The generated scatter and trend graphs physically confirm these periods correspond to severe deviations from normal operating bands.

**Table of Identified Abnormal Periods:**

| Start Date | End Date | Duration (Hours) | Fault Type | Main Signal |
|:---|:---|---:|:---|:---|
| 2017-02-24 | 2017-02-25 | 32.0 | C: Draft instability | cone_draft_std |
| 2017-02-27 | 2017-03-01 | 63.0 | C: Draft instability | cone_draft_std |
| 2017-03-29 | 2017-03-31 | 46.0 | C: Draft instability | cone_draft_std |
| 2017-10-25 | 2017-10-26 | 17.0 | C: Draft instability | cone_draft_std |
| 2018-01-27 | 2018-01-28 | 32.0 | D: Pressure-drop change | d_out_in |
| 2018-03-08 | 2018-03-20 | 295.0 | A: Build-up / blockage | dT |
| 2018-04-15 | 2018-04-16 | 36.0 | A: Build-up / blockage | dT |
| 2018-04-19 | 2018-04-29 | 237.0 | A: Build-up / blockage | dT |
| 2018-05-04 | 2018-05-07 | 66.0 | A: Build-up / blockage | dT |
| 2018-05-08 | 2018-05-08 | 16.0 | A: Build-up / blockage | dT |
| 2018-05-09 | 2018-05-11 | 43.0 | A: Build-up / blockage | dT |
| 2018-05-17 | 2018-05-18 | 36.0 | A: Build-up / blockage | dT |
| 2018-05-20 | 2018-05-21 | 34.0 | A: Build-up / blockage | dT |
| 2018-09-18 | 2018-09-19 | 18.0 | B: Outlet over-temperature | dT |
| 2018-09-21 | 2018-09-21 | 15.0 | C: Draft instability | cone_draft_std |
| 2019-02-02 | 2019-02-04 | 47.0 | A: Build-up / blockage | dT |
| 2019-02-08 | 2019-02-13 | 122.0 | A: Build-up / blockage | dT |
| 2019-03-19 | 2019-03-28 | 219.0 | A: Build-up / blockage | dT |
| 2019-04-24 | 2019-04-24 | 13.0 | A: Build-up / blockage | dT |
| 2019-08-23 | 2019-08-24 | 26.0 | A: Build-up / blockage | dT |
| 2019-11-13 | 2019-11-15 | 52.0 | A: Build-up / blockage | dT |
| 2019-12-26 | 2019-12-27 | 14.0 | C: Draft instability | cone_draft_std |

---

### Conclusion

In plain terms, we successfully built an "early warning system" for the cyclone using basic physics and statistics. 

We found that about 85% of all plant issues are caused by material building up inside the cyclone, blocking the hot gas flow. This buildup doesn't happen instantly; it grows over weeks before finally collapsing. Our charts clearly show the warning signs appearing days before the problem becomes critical. By tracking how these sensors relate to each other—rather than just waiting for one to hit a standard redline alarm—plant operators can now spot blockages early, schedule cleanings before a full shutdown is required, and save significant downtime.

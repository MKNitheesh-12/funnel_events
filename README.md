# Funnel Events Analysis

A Python/pandas analysis of a user signup-to-purchase funnel, built from raw event-level data. The notebook cleans event logs, builds a stage-by-stage conversion funnel, flags the biggest drop-off points, measures time-to-convert between stages, and breaks results down by user cohort.

## Overview

The dataset tracks users moving through five funnel stages:

1. `visited_site`
2. `signup_started`
3. `details_filled`
4. `email_verified`
5. `purchase_completed`

Each row in the source data is a single event: a `user_id`, a `step`, and a `timestamp`. Users can appear multiple times per step (duplicate/repeat events), and not every user progresses through every stage — the analysis is built to handle both of these realities.

## Dataset

**File:** `funnel_events_sample.csv`

| Column      | Type          | Description                          |
|-------------|---------------|---------------------------------------|
| `user_id`   | string        | Unique user identifier (e.g. `U001`)  |
| `step`      | string        | Funnel stage name                    |
| `timestamp` | datetime      | When the event occurred              |

- 542 raw events across 200 unique users.
- Events are not guaranteed to be in order or deduplicated — the notebook sorts by `step_rank` and timestamp, then keeps each user's **earliest** occurrence of each step.

## What the notebook does

### 1. Data cleaning
- Parses `timestamp` into proper datetime objects.
- Maps each `step` to a fixed sequence (`step_order`) so stages can be sorted and compared correctly, even though they don't appear in order in the raw log.
- Deduplicates repeated events per user/step, keeping the earliest timestamp as the "true" moment a user reached that stage.

### 2. Funnel summary table
For each stage, computes:
- **Unique Users** reaching that stage
- **% of Top** — percentage relative to the first stage (`visited_site`)
- **Stage-to-Stage Conversion Rate** — % of users from the previous stage who advanced
- **Users Dropped Off** — raw count lost vs. the previous stage
- **Drop-off Rate** — % lost vs. the previous stage

### 3. Automated drop-off flagging
Identifies the biggest drop-off stage two ways, since volume and percentage loss can point to different stages:
- By **percentage loss** (steepest relative drop)
- By **user volume** (largest absolute number of users lost)

### 4. Time-to-convert analysis
For users who completed each stage transition, computes the average (and median) time in minutes between:
- `visited_site → signup_started`
- `signup_started → details_filled`
- `signup_started → purchase_completed` (direct buyers)

Only pairs where a user has valid timestamps for both stages are included.

### 5. Cohort / segment breakdown
Splits users into two cohorts by user ID range (`U001–U100` vs. `U101–U200`) and compares stage counts side by side, surfacing behavioral differences between the groups (e.g. one cohort converting much better at `signup_started`).

### 6. Visualization
Generates a bar chart of unique users per stage (`funnel_chart.png`) showing the funnel shape at a glance.

## Key findings (from the sample data)

| Stage | Unique Users | % of Top | Conversion Rate | Drop-off Rate |
|---|---|---|---|---|
| visited_site | 200 | 100.0% | 100.0% | — |
| signup_started | 150 | 75.0% | 75.0% | 25.0% |
| details_filled | 96 | 48.0% | 64.0% | 36.0% |
| email_verified | 52 | 26.0% | 54.2% | 45.8% |
| purchase_completed | 44 | 22.0% | ~84.6% | ~15.4% |

- **Biggest drop-off by percentage:** `email_verified` (~45.8% of users who filled details never verified their email).
- **Biggest drop-off by volume:** `details_filled` (54 users lost between signup and filling details).
- Cohort breakdown shows a meaningful split: one cohort stalls hard at `signup_started`, while the other struggles more to get from `visited_site` to `signup_started` — worth investigating as separate UX/onboarding issues rather than one universal funnel problem.

> Note: some out-of-order event timestamps exist in the raw log (e.g. a few users show `signup_started` before `visited_site`, or `details_filled` before `signup_started`). The cleaning step handles this by keeping the earliest event per stage rather than assuming strict chronological order across stages.

## Repository structure

```
.
├── funnel_events_sample.ipynb   # Main analysis notebook
├── funnel_events_sample.csv     # Raw event-level input data
├── funnel_chart.png             # Output: bar chart of users per funnel stage
└── README.md
```

## Requirements

```
pandas
numpy
matplotlib
```

Install with:

```bash
pip install pandas numpy matplotlib
```

## Usage

1. Place `funnel_events_sample.csv` in the same directory as the notebook.
2. Open and run `funnel_events_sample.ipynb` top to bottom (Jupyter, JupyterLab, VS Code, or Colab).
3. Review the printed funnel summary table and drop-off flags in the notebook output.
4. Find the generated chart at `funnel_chart.png`.

## Possible next steps

- Replace the fixed `step_order` list with a config file so the funnel can be reused for other event schemas.
- Add statistical significance testing to cohort comparisons (sample sizes here are small).
- Parameterize the cohort split (currently a hardcoded user-ID cutoff) to support real segmentation attributes (e.g. acquisition channel, device type).
- Export the summary table and chart automatically to a `reports/` folder with a timestamp.

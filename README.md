# Funnel Events Analysis

A Python/pandas notebook that analyzes user progression through a signup funnel — from site visit to purchase completion — and surfaces where and why users drop off.

## What it does

Given a raw event log of user actions, the notebook:

- **Cleans and orders events** — parses timestamps, maps each event to its position in the funnel, and deduplicates repeat events per user/step (keeping the earliest occurrence).
- **Builds a funnel summary table** — unique users per stage, % of top-of-funnel, stage-to-stage conversion rate, and drop-off count/rate.
- **Flags the biggest drop-off stage** — both by percentage lost and by raw user volume, since these can point to different stages.
- **Calculates time-to-convert** — average and median minutes between consecutive stages (e.g. visit → signup, signup → purchase for direct buyers).
- **Breaks down results by cohort/segment** — compares funnel behavior between user ID ranges to help spot cohort-specific issues.
- **Charts the funnel** — a labeled bar chart of unique users per stage, saved as a PNG.

## Funnel stages

```
visited_site → signup_started → details_filled → email_verified → purchase_completed
```

## Data format

Input is a CSV (`funnel_events_sample.csv`) with one row per user event:

| column      | type   | description                          |
|-------------|--------|--------------------------------------|
| `user_id`   | string | unique user identifier (e.g. `U001`) |
| `step`      | string | funnel stage name                    |
| `timestamp` | string | event time (`YYYY-MM-DD HH:MM:SS`)   |

## Requirements

- Python 3.8+
- pandas
- numpy
- matplotlib

```bash
pip install pandas numpy matplotlib
```

## Usage

1. Place `funnel_events_sample.csv` in the working directory.
2. Run the notebook cells top to bottom (or export/run as a script).
3. Review console output for the summary table, drop-off flags, and time-to-convert metrics.
4. Find the funnel chart at `funnel_chart.png`.

## Sample output

```
             Stage  Unique Users  % of Top  Conversion Rate (%)  Users Dropped Off  Drop-off Rate (%)
      visited_site           200     100.0                100.0                  0               0.00
    signup_started           150      75.0                 75.0                 50              25.00
    details_filled            96      48.0                 64.0                 54              36.00
    email_verified            52      26.0                54.17                 44              45.83
purchase_completed            44      22.0                84.62                  8              15.38

Largest Drop-off by %: email_verified (45.83% loss from previous stage)
Largest Drop-off by Volume: details_filled (54 users lost)
```

## Notes

- Drop-off can be measured two ways — by **percentage lost** relative to the previous stage, or by **raw user count** lost — and the notebook reports both, since they can disagree (as in the sample data above).
- Time-to-convert metrics only include users who reached *both* stages in a given pair; users who skip a stage are excluded from that pair's average.
- Cohort/segment splits in this sample use user ID ranges as a stand-in for real segment fields (e.g. acquisition channel, plan type) — swap in your own segment column as needed.

## License

MIT

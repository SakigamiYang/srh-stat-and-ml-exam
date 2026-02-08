# Data Contract: `diabetes-event-window-features` (Event-Window Supervised Table)

**Purpose:** One row per **blood glucose measurement event**, transforming irregular event sequences 
into fixed-length feature windows for **supervised learning**, with the objective of 
predicting the **next blood glucose value**.

## Source dataset

* **Input files:** `data-01` … `data-70`
* **Input record schema (per line):** `MM-DD-YYYY<TAB>HH:MM<TAB>code<TAB>value`
* **Codes:** see mapping (33–35 insulin, 48/57/58–64 glucose, 65–72 events)

## Prediction task

* **Task type:** regression or discretized classification
* **Target:** next blood glucose measurement value
* **Prediction horizon:** next glucose event in time (same patient-file)

---

## Grain and keys

* **Grain:** event-level (per glucose measurement)
* **Primary key:** (`id`, `event_time`)
* **Row meaning:** one supervised sample constructed at a glucose measurement time, 
* using only prior events within a fixed lookback window.

---

## Windowing logic

* **Index event:** a glucose measurement at time `t`
* **Lookback window:** events occurring in `(t − Δ, t]`
* **Window size:** configurable (e.g. 2h, 4h, 6h)
* **Causality constraint:** no information from future events (`> t`) is used

---

## Contract table (core fields + families)

| Field                |     Type | Nullable | Description                              | Derivation                 |
|----------------------|---------:|:--------:|------------------------------------------|----------------------------|
| `id`                 |   string |    no    | File identifier (`data-01`…`data-70`)    | From filename              |
| `event_time`         | datetime |    no    | Timestamp of current glucose event       | Parsed from record         |
| `target_time`        | datetime |    no    | Timestamp of next glucose event          | Next glucose record        |
| `delta_t_minutes`    |      int |    no    | Minutes between current and next glucose | `target_time - event_time` |
| `y_next_glucose`     |    float |    no    | Next blood glucose value                 | From next glucose record   |
| `y_next_glucose_bin` |      int |   yes    | Discretized glucose class (optional)     | Threshold-based            |

---

## Glucose history feature family

Computed from glucose measurements within the lookback window.

| Field                        |  Type | Nullable | Description                      |
|------------------------------|------:|:--------:|----------------------------------|
| `bg_last_value`              | float |   yes    | Most recent glucose before `t`   |
| `bg_window_count`            |   int |    no    | #glucose measurements in window  |
| `bg_window_mean/std/min/max` | float |   yes    | Summary statistics               |
| `bg_window_trend`            | float |   yes    | Linear trend (slope) over window |
| `time_since_last_bg`         |   int |   yes    | Minutes since previous glucose   |

---

## Insulin feature family

Computed from insulin events (codes 33–35) within the lookback window.

| Field                 |  Type | Nullable | Description                |
|-----------------------|------:|:--------:|----------------------------|
| `ins_window_count`    |   int |    no    | #insulin events            |
| `ins_window_sum`      | float |   yes    | Total insulin dose         |
| `ins_window_mean/max` | float |   yes    | Dose statistics            |
| `time_since_last_ins` |   int |   yes    | Minutes since last insulin |

---

## Event feature family (counts + flags)

Computed from non-glucose, non-insulin events (codes 65–72).

| Field                   | Type | Nullable | Description                     |
|-------------------------|-----:|:--------:|---------------------------------|
| `meal_events_count`     |  int |    no    | Meal-related events (66–68)     |
| `exercise_events_count` |  int |    no    | Exercise-related events (69–71) |
| `hypo_event_flag`       | bool |    no    | Hypoglycemic symptom occurred   |
| `special_event_flag`    | bool |    no    | Special event occurred          |

---

## Temporal context features

| Field                     | Type | Nullable | Description                          |
|---------------------------|-----:|:--------:|--------------------------------------|
| `hour_of_day`             |  int |    no    | Hour of `event_time` (0–23)          |
| `day_of_week`             |  int |    no    | Monday=0…Sunday=6                    |
| `is_weekend`              | bool |    no    | Weekend indicator                    |
| `is_paper_like_time_flag` | bool |    no    | Timestamp matches paper-record slots |

---

## Data quality and filtering rules

* Samples without a subsequent glucose measurement are dropped
* Overlapping windows are allowed
* All features must be derived strictly from events at or before `event_time`
* Missing numeric features are imputed deterministically (e.g. 0 or sentinel)
* Train/test splits must be grouped by `id` to avoid patient-level leakage

---

## Intended use

This dataset is designed for supervised models that learn short-term glucose dynamics from recent behavioral and physiological context, such as linear regression, tree-based models, or sequence-aware architectures operating on fixed-length feature windows.

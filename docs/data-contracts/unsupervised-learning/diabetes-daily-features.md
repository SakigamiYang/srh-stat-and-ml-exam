# Data Contract: `diabetes_daily_features` (Wide Daily Feature Table)

**Purpose:** One row per **patient-file (`id`) × calendar day (`date`)**, converting the original event log (code/value) into a feature matrix suitable for **unsupervised learning** (clustering, anomaly detection, similarity search).

## Source dataset

* **Input files:** `data-01` … `data-70`
* **Input record schema (per line):** `MM-DD-YYYY<TAB>HH:MM<TAB>code<TAB>value`
* **Codes:** see mapping (33–35 insulin, 48/57/58–64 glucose, 65–72 events)

## Grain and keys

* **Grain:** daily
* **Primary key:** (`id`, `date`)
* **Row meaning:** all events for one patient-file on one day, aggregated into features.

---

## Contract table (core fields + families)

| Field / Family              |   Type | Nullable | Description                              | Derivation                      |
|-----------------------------|-------:|:--------:|------------------------------------------|---------------------------------|
| `id`                        | string |    no    | File identifier (`data-01`…`data-70`)    | From filename                   |
| `date`                      |   date |    no    | Calendar day                             | Parsed from record date         |
| `dow`                       |    int |    no    | Day of week, Monday=0..Sunday=6          | `date.weekday()`                |
| `is_weekend`                |   bool |    no    | Weekend flag                             | `dow in {5,6}`                  |
| `n_events`                  |    int |    no    | Number of raw records that day           | Count of rows for (`id`,`date`) |
| `n_unique_timestamps`       |    int |    no    | Unique `HH:MM` count within day          | `nunique(time)`                 |
| `first_time_minutes`        |    int |   yes    | Earliest event minute-of-day             | `min(time)` → minutes           |
| `last_time_minutes`         |    int |   yes    | Latest event minute-of-day               | `max(time)` → minutes           |
| `active_span_minutes`       |    int |   yes    | Activity span in minutes                 | `last-first`                    |
| `events_morning_count`      |    int |    no    | #records in 05:00–11:59                  | time bucket                     |
| `events_afternoon_count`    |    int |    no    | #records in 12:00–17:59                  | time bucket                     |
| `events_evening_count`      |    int |    no    | #records in 18:00–23:59                  | time bucket                     |
| `events_night_count`        |    int |    no    | #records in 00:00–04:59                  | time bucket                     |
| `has_paper_like_times_flag` |   bool |    no    | Many events at {08:00,12:00,18:00,22:00} | share ≥ threshold               |
| `missing_bg_flag`           |   bool |    no    | No glucose measurements that day         | bg codes absent                 |
| `missing_insulin_flag`      |   bool |    no    | No insulin doses that day                | insulin codes absent            |

### Glucose feature family (per code)

Applies to codes: **48, 57, 58, 59, 60, 61, 62, 63, 64**
Each code produces these numeric fields (nullable when code absent that day):

| Field pattern               |  Type | Nullable | Description                          |
|-----------------------------|------:|:--------:|--------------------------------------|
| `bg_{code}_count`           |   int |    no    | #measurements for that code that day |
| `bg_{code}_min/max`         | float |   yes    | Min/max glucose value                |
| `bg_{code}_mean/median/std` | float |   yes    | Summary statistics                   |
| `bg_{code}_first/last`      | float |   yes    | First/last value by time order       |
| `bg_{code}_range`           | float |   yes    | `max - min`                          |

### Glucose “all” family (merged across all glucose codes)

| Field                            |  Type | Nullable | Description                                    |
|----------------------------------|------:|:--------:|------------------------------------------------|
| `bg_all_count`                   |   int |    no    | Total glucose measurement count (all bg codes) |
| `bg_all_min/max/mean/median/std` | float |   yes    | Stats across all bg values                     |
| `bg_all_range`                   | float |   yes    | `max - min`                                    |

### Insulin feature family (per code)

Applies to codes: **33, 34, 35**

| Field pattern                        |  Type | Nullable | Description             |
|--------------------------------------|------:|:--------:|-------------------------|
| `ins_{code}_count`                   |   int |    no    | #doses that day         |
| `ins_{code}_sum`                     | float |   yes    | Total dose that day     |
| `ins_{code}_min/max/mean/median/std` | float |   yes    | Dose stats              |
| `ins_{code}_first/last`              | float |   yes    | First/last dose by time |
| `ins_{code}_range`                   | float |   yes    | `max - min`             |

### Insulin “all” family

| Field              |  Type | Nullable | Description                          |
|--------------------|------:|:--------:|--------------------------------------|
| `ins_all_count`    |   int |    no    | Total insulin dose events (33/34/35) |
| `ins_all_sum`      | float |   yes    | Total insulin dose (33/34/35)        |
| `ins_all_mean/std` | float |   yes    | Stats across insulin values          |

### Event feature family (count + flag)

Applies to codes: **65–72** (symptoms/meals/exercise/special)

| Field pattern      | Type | Nullable | Description                   |
|--------------------|-----:|:--------:|-------------------------------|
| `evt_{code}_count` |  int |    no    | Count of occurrences that day |
| `evt_{code}_flag`  | bool |    no    | Whether it happened that day  |

Derived rollups:

* `meal_events_count` (66+67+68), `meal_more_flag` (67), `meal_less_flag` (68)
* `exercise_events_count` (69+70+71), `exercise_more_flag` (70), `exercise_less_flag` (71)
* `hypo_flag` (65), `special_flag` (72)

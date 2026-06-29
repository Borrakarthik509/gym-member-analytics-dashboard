# 📖 Data Dictionary

[← Back to README](../README.md)

> Complete column definitions, data types, and table descriptions for every table in the Gym Member Analytics Dashboard data model.

---

## Table of Contents

- [Gym Dataset (Fact Table)](#table-gym-dataset-fact-table)
- [Measure_table (Measure Island)](#table-measure_table-measure-island)
- [Mood (Dimension)](#table-mood-dimension)
- [Gender (Dimension)](#table-gender-dimension)
- [Steps Bucket (Dimension)](#table-steps-bucket-dimension)
- [Steps Category (Dimension)](#table-steps-category-dimension)
- [Group Lesson (Dimension)](#table-group-lesson-dimension)
- [Birth Data (Dimension)](#table-birth-data-dimension)
- [Exercise (Dimension)](#table-exercise-dimension)

---

## Table: Gym Dataset (Fact Table)

The central fact table containing one row per gym member with their activity metrics, membership details, and trainer assignments.

| Column | Data Type | Description | Example Values |
|--------|-----------|-------------|----------------|
| `id` | Integer | Unique member identifier | `1001`, `1002` |
| `name_personal_trainer` | Text | Full name of the assigned personal trainer | `"Mike"`, `"Chantal"` |
| `abonoment_type` | Text | Membership tier | `"Premium"`, `"Standard"` |
| `mood` | Integer / Text | Raw post-workout mood value | `1`–`5` or text label |
| `steps` | Integer | Daily step count recorded | `3000`–`18000` |
| `calories_burned` | Decimal | Calories burned during session | `150.0`–`900.0` |
| `active_minutes` | Integer | Duration of active workout in minutes | `20`–`120` |
| `visits` | Integer | Number of gym visits per week | `1`–`5` |
| `age` | Integer | Member age in years | `22`–`50` |
| `session_time` | Integer | Average session duration in minutes | `40`–`120` |
| `gender` | Text | Member gender | `"Male"`, `"Female"` |
| `fav_group_lesson` | Text | Preferred group class | `"HIIT"`, `"Yoga"`, `"Spinning"` |
| `birthday` | Date | Member date of birth | `1978-10-03` |

---

## Table: Measure_table (Measure Island)

A disconnected table containing all DAX measures — best practice for separating business logic from raw data columns.

| Measure | Return Type | Description |
|---------|-------------|-------------|
| `avg_visit` | Decimal | Average number of gym visits per member across the filtered context |
| `avg_age` | Decimal | Average age of members in the filtered context |
| `avg_time` | Decimal | Average session duration (minutes) in the filtered context |

> **Why a measure island?** Storing measures in a disconnected table keeps the fact table's field list clean (only raw columns), centralises all business logic in one location, and simplifies future refactoring.

---

## Table: Mood (Dimension)

Maps raw mood identifiers to human-readable labels for chart display.

| Column | Description |
|--------|-------------|
| `Mood` | Raw mood identifier — joins to `Gym Dataset[mood]` |
| `Mood Display` | Human-readable mood label (e.g., `"Happy"`, `"Neutral"`, `"Tired"`) |
| `mood` | Alternate mood column (may be used as secondary join key) |
| `Links` | Slicer binding field — used by advanced button slicer to enable custom display |

---

## Table: Gender (Dimension)

Provides display labels for the gender filter slicer.

| Column | Description |
|--------|-------------|
| `gender label` | Display label for gender filter (`"Male"`, `"Female"`, `"Other"`) |
| `Links` | Slicer binding field |

---

## Table: Steps Bucket (Dimension)

Bucketed step ranges used on the x-axis of the Steps vs Calories line chart.

| Column | Description |
|--------|-------------|
| `steps bucket` | Bucketed step range label (e.g., `"0–5k"`, `"5k–10k"`, `"10k+"`) |
| `Links` | Slicer binding field |

---

## Table: Steps Category (Dimension)

Broader activity tier labels used as a report-level slicer filter.

| Column | Description |
|--------|-------------|
| `Steps Category` | Activity intensity label (e.g., `"Sedentary"`, `"Active"`, `"Highly Active"`) |
| `Link` | Slicer binding field |

> **Why two step tables?** `Steps bucket` provides granular numeric ranges for chart axes, while `Steps Category` provides broader tier labels for filtering. Keeping them separate allows independent sort orders and display logic.

---

## Table: Group Lesson (Dimension)

Lists available group exercise classes for the left sidebar slicer.

| Column | Description |
|--------|-------------|
| `fav_group_lesson` | Name of the group class (e.g., `"Yoga"`, `"Spinning"`, `"HIIT"`) |
| `Links` | Slicer binding field |

---

## Table: Birth Data (Dimension)

Member date-of-birth data used at the month/year hierarchy level in the combo chart.

| Column | Description |
|--------|-------------|
| `birthday` | Member date of birth — used at month/year hierarchy level in the combo chart |

---

## Table: Exercise (Dimension)

Referenced in the data model; specific columns are not surfaced in report visuals. Used to classify exercise types associated with activity records.

---

## Relationships Overview

| From (Dimension) | To (Fact) | Join Column | Direction |
|-------------------|-----------|-------------|-----------|
| `Gender` | `Gym Dataset` | `gender` | Single |
| `Mood` | `Gym Dataset` | `mood` | Single |
| `Exercise` | `Gym Dataset` | exercise key | Single |
| `Steps bucket` | `Gym Dataset` | `steps bucket` | Single |
| `Steps Category` | `Gym Dataset` | `Steps Category` | Single |
| `Group lesson` | `Gym Dataset` | `fav_group_lesson` | Single |
| `Birth data` | `Gym Dataset` | `birthday` | Single |
| `Measure_table` | *(disconnected)* | — | — |

> All dimension-to-fact joins are **many-to-one** with **single-direction** filtering. The `Links` columns in dimension tables serve as slicer anchors for advanced button slicers, enabling cross-filtering without affecting the fact table's row count.

# ASL 20 Race vs Map Analysis Pipeline

## 📌 Overview

This project builds a data pipeline to analyze **win rates by race and map** using mock ASL 20 tournament data.

The pipeline simulates a real-world data engineering workflow using:

* **Mock JSON data** (simulating API ingestion)
* **dbt** for data transformations
* **(Optional) Airflow** for orchestration

---

## 🎯 Goal

Answer questions like:

* What is the win rate of each race on each map?
* How do matchups (e.g., Zerg vs Protoss) perform on specific maps?
* (Future) Player-specific performance by race and map

---

## 🗂️ Project Structure

```
project-root/
│
├── data/
│   ├── players.json
│   ├── matches.json
│   ├── games.json
│   └── player_match_race.json
│
├── dbt/
│   ├── models/
│   │   ├── staging/
│   │   ├── intermediate/
│   │   └── marts/
│   └── dbt_project.yml
│
├── dags/                 # (optional for Airflow later)
│
└── README.md
```

---

## 📊 Data Model

### 1. `players.json`

Contains player-level information.

Fields:

* `player_id`
* `race`

---

### 2. `matches.json`

Represents matchups between two players.

Fields:

* `match_id`
* `player_1`
* `player_2`
* `winner`
* `round`

---

### 3. `games.json` ⭐ (most important)

Each row represents a **single game on a specific map**.

Fields:

* `game_id`
* `match_id`
* `map`
* `winner`

---

### 4. `player_match_race.json` (optional)

Allows race to vary per match (future-proofing).

Fields:

* `match_id`
* `player_id`
* `race`

---

## 🧠 Key Transformation Concept

Raw match data is **wide**:

```
match_id | player_1 | player_2 | winner
```

We transform it into a **long format**:

```
game_id | player_id | race | map | is_winner
```

Each game produces **2 rows (one per player)**.

This enables simple aggregation for analytics.

---

## 🔄 Transformation Flow (dbt)

### 🟢 Staging Layer

* Load raw JSON into tables
* Clean column names
* Ensure consistent types

Models:

* `stg_players`
* `stg_matches`
* `stg_games`

---

### 🟡 Intermediate Layer

* Join matches + games + players
* Expand into **player-level rows**

Model:

* `int_player_games`

---

### 🔵 Mart Layer

Final analytics-ready table.

Model:

* `fact_player_games`

Fields:

* `game_id`
* `player_id`
* `race`
* `map`
* `is_winner`

---

## 📈 Example Query

### Win rate by race and map

```sql
SELECT
  map,
  race,
  AVG(is_winner) AS win_rate
FROM fact_player_games
GROUP BY map, race
ORDER BY map, race;
```

---

## 🚀 Getting Started

### 1. Save Mock Data

Place JSON files into:

```
data/
```

---

### 2. Load Data into Database

Options:

* Postgres (recommended)
* DuckDB (simplest)
* BigQuery / Snowflake (if preferred)

---

### 3. Set Up dbt

Initialize dbt project:

```
dbt init asl_analysis
```

Configure your database connection in:

```
profiles.yml
```

---

### 4. Build Models

Run:

```
dbt run
```

Then:

```
dbt test
```

---

## 🔥 Future Enhancements

* Add **series-level data (Bo3, Bo5)**
* Handle **multiple tournaments (ASL 19, 20, 21...)**
* Add **incremental models**
* Integrate **Airflow DAG**
* Build dashboard (e.g., Tableau, Superset)

---

## ⚠️ Notes & Assumptions

* Player race is assumed **static per match** (sufficient for ASL)
* Map names must be **consistent** across all records
* Each game has exactly **one winner**

---

## 💡 Why This Project Matters

This project demonstrates:

* Handling semi-structured data (JSON)
* Data normalization
* Fact/dimension modeling
* Analytical query design
* dbt best practices

---

## 🏁 Next Step

Implement the **`int_player_games` dbt model**:

This is the most important transformation in the pipeline.

Once that works, everything else becomes straightforward.

---

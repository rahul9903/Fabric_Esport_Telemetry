# Project Link
https://app.fabric.microsoft.com/links/XAgNBeEoFY?ctid=b1ecfa5f-bf5c-4d47-8691-c39477ec0ac6&pbi_source=linkShare
<img width="1284" height="715" alt="Screenshot_10-11-2026_145846_app fabric microsoft com" src="https://github.com/user-attachments/assets/fef255dc-3e1e-41dc-bae5-39eee0ce2360" />


# Real-Time Esports Match Telemetry & Combat Momentum Intelligence

An end-to-end streaming data engineering and spatial combat analytics platform built natively on **Microsoft Fabric**. This solution streams high-frequency esports player telemetry (tracking spatial coordinates, round economy, active hit points, utility usage, and elimination events), ingests events simultaneously into real-time KQL databases and Lakehouse Delta storage, transforms match telemetry across a Medallion Delta pipeline (PySpark), and serves live broadcast combat analytics in Power BI via **Direct Lake** mode with zero data refresh latency.

---

## 1. System Architecture & End-to-End Flow

The platform implements a unified streaming and Lakehouse architecture across Microsoft Fabric:

1. **Telemetry Producer (Stream Source):** A telemetry simulator generates millisecond-level combat event streams representing high-stakes tactical shooter competition (Fnatic vs. Sentinels). Events capture player positional coordinates (coord_x, coord_y), player hit points (hp_remaining), loadout purchase values (current_loadout_value), opponent surviving headcounts (opponent_alive_count), and granular action types (kill, damage_dealt, utility_used, bomb_plant).
2. **Fabric Real-Time Ingestion (Eventstream):** An Eventstream (`es_match_telemetry`) ingests live streaming telemetry batches and routes the payload concurrently into two parallel processing engines:
   * **Real-Time Storage Path:** Feeds directly into a Fabric Eventhouse (`kql_esports_live`) and KQL Database for instant sliding-window queries and automated condition monitoring (`Activator`).
   * **Lakehouse Storage Path:** Streams raw append-only records directly into the Lakehouse (`lh_esports_analytics`) as OneLake Delta tables.
3. **KQL Real-Time Combat Monitoring:** A KQL Queryset (`qs_esports_realtime`) queries recent rolling five-minute event windows, calculating live total loadout investments, team average health, active alive player headcounts, and rapid elimination runs by round and team.
4. **Lakehouse Medallion Processing:** An automated Apache Spark notebook (`nb_transform_telemetry`) reads raw streaming Delta tables, deduplicates and normalizes schemas in Silver, and calculates round-by-round combat momentum features in Gold.
5. **Direct Lake Semantic Layer:** A Power BI Semantic Model (`sm_esports_directlake`) connects directly to OneLake Delta Parquet files using **Direct Lake** mode, serving aggregated features and spatial coordinates at VertiPaq speeds without scheduled data import refreshes.
6. **Executive Power BI Presentation:** An interactive broadcast intelligence dashboard (`rpt_esports_match_telemetry`) visualizes player combat spatial distributions, elimination heatmaps, round-by-round combat power momentum indices, and player defensive survival ratios.

---

## 2. Lakehouse Medallion Data Engineering

The Lakehouse pipeline (`lh_esports_analytics`) organizes data into three distinct layers to ensure schema consistency, deduplication, and high-performance analytical aggregation:

### Bronze Layer (`bronze_match_events`)
* Serves as the immutable, raw landing zone for incoming streaming telemetry records.
* Retains unedited match payloads including `match_id`, `round_num`, `timestamp`, `team`, `player_id`, `event_type`, `coord_x`, `coord_y`, `hp_remaining`, `current_loadout_value`, and `opponent_alive_count`.

### Silver Layer (`silver_match_events`)
* Cleanses, validates, and normalizes the raw Bronze data.
* Converts string timestamps into typed event time metadata (`event_time`) and enforces strict numeric casting on spatial coordinates, player hit points, economy values, and opponent counts.
* Executes multi-column deduplication across composite keys (`match_id`, `event_time`, `player_id`, `event_type`) to eliminate duplicated telemetry packets caused by streaming re-transmissions.

### Gold Layer (`gold_round_features`)
* A curated feature store built for dimensional modeling, round evaluation, and power index scoring.
* Aggregates team performance at the round grain (`match_id`, `round_num`, `team`).
* Evaluates economic capability by calculating average team loadout valuation per round.
* Tracks team aggression and execution by tallying round-level elimination totals and surviving health points.
* Synthesizes a composite **Combat Power Index** for every round, weighting sixty percent of a team's economic loadout valuation against forty percent of their remaining health reserves to quantify real-time match momentum.

---

## 3. Direct Lake Semantic Modeling & Relationships

The semantic model (`sm_esports_directlake`) links the curated Delta tables in OneLake directly to Power BI visuals:

### Data Model Topology
* **Fact Table (`silver_match_events`):** Contains granular player event coordinates and action metrics for spatial plotting and roster filtering.
* **Feature Dimension Table (`gold_round_features`):** Contains round-level summaries, combat power indices, and economic metrics.
* **Model Relationship:** Connected on a one-to-many relationship from `gold_round_features` to `silver_match_events` across team and round context, enabling seamless cross-filtering from macro round performance down to micro player actions.

### Core Analytical Measures
* **Total Eliminations:** Aggregates confirmed kill events across all competing rosters and rounds.
* **Momentum Spread:** Calculates the mathematical divergence between opposing combat power indices, highlighting whether Fnatic or Sentinels holds economic and battlefield superiority in any given round.
* **Opponent Pressure Index:** Measures the surviving opponent density during active skirmishes to identify high-impact defensive holds and clutch round conversions.

---

## 4. Power BI Presentation Layer (`rpt_esports_match_telemetry`)

The visual console employs a broadcast-style dark theme (`#0F111A`) tailored for tournament observers, analysts, and coaching staff:

### Visual Layout & Insights
1. **Player Combat & Elimination Map (Scatter Plot - Top Left):**
   * **X-Axis:** `coord_x` (Continuous spatial coordinates 0 to 400).
   * **Y-Axis:** `coord_y` (Continuous spatial coordinates 0 to 400).
   * **Legend:** `team` (Fnatic in Vibrant Orange vs. Sentinels in Electric Cyan).
   * **Details:** `player_id` and `event_type`.
   * **Function:** Visualizes territorial combat control, choke-point engagements, flank routes, and combat densities across the arena.
2. **Key Metric Cards (Top Right):**
   * **Momentum Spread KPI Card:** Displays the net combat power advantage between teams.
   * **Total Eliminations KPI Card:** Displays total match-wide frag volume.
3. **Round Combat Momentum Index (Dual-Line Trend Chart - Bottom Left):**
   * **X-Axis:** `round_num` (Rounds 1 through 15).
   * **Y-Axis:** `combat_power_index`.
   * **Legend:** `team`.
   * **Function:** Tracks swings in economic control, buy phases (eco rounds vs. full rifle buys), round resets, and winning streaks across the entire match timeline.
4. **Player Survival & Skirmish Engagement Board (Horizontal Bar Chart - Bottom Right):**
   * **Y-Axis:** `team` and `player_id` roster hierarchy.
   * **X-Axis:** `opponent_alive_count`.
   * **Function:** Highlights which players consistently survive into late-round scenarios and endure hostile 5vX combat pressure.

---

## 5. Fabric Platform Artifacts Reference

* **Eventstream (`es_match_telemetry`):** Central streaming hub routing live player events to KQL and Lakehouse destinations.
* **Eventhouse & KQL Database (`kql_esports_live`):** Low-latency telemetry database for streaming ingestion and immediate time-series analytics.
* **KQL Queryset (`qs_esports_realtime`):** Real-time queries computing sliding-window loadout averages, health stats, and eliminations.
* **Data Activator (`Activator 2026-11-10`):** Automated alert triggers monitoring sudden combat momentum collapse or rapid multi-kill sequences.
* **Lakehouse (`lh_esports_analytics`):** Unified OneLake storage engine housing Bronze, Silver, and Gold Delta tables.
* **Spark Notebook (`nb_transform_telemetry`):** PySpark Medallion pipeline handling deduplication, type casting, and feature calculation.
* **Semantic Model (`sm_esports_directlake`):** Direct Lake model streaming OneLake Delta parquet data straight into Power BI memory.
* **Power BI Report (`rpt_esports_match_telemetry`):** Dark-themed broadcast dashboard displaying tactical maps, momentum curves, and player combat analytics.

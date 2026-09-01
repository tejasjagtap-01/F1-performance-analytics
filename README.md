<img width="1482" height="728" alt="git post" src="https://github.com/user-attachments/assets/1997a5f3-6f7a-4cf9-9681-8160fb4c5194" />




#  Section 1: Python EDA & Data Transformation Summary

Processed 14 relational tables (1950–2024, 1,100+ Grands Prix) using Python, Pandas, Matplotlib, and Seaborn to clean historical telemetry, engineer racecraft metrics, and generate `fact_race_results.csv` for downstream SQL and Power BI modeling.

---

### Pipeline & Feature Engineering
* **Relational Merges:** Joined `results.csv`, `drivers.csv`, `constructors.csv`, and `status.csv` to map driver identities and decode non-finishing metadata.
* **Racecraft Metric:** Calculated on-track overtakes via {positions\_gained} = \{grid} - \{positionOrder}.
* **Status Normalization:** Flagged retirements (`is_dnf = 1`) while isolating specific mechanical and incident failure types.
* **Fact Export:** Exported the consolidated dataset as `fact_race_results.csv`.

---

### Core Analyses & Code Implementations

| Analysis Area | Target Logic | Primary Visualization Code |
| :--- | :--- | :--- |
| **Driver All-Time Wins** | Filtered `positionOrder == 1` grouped by driver | `sns.barplot(x=top_winners.values, y=top_winners.index, color='steelblue')` |
| **Constructor Wins** | Filtered `positionOrder == 1` grouped by constructor | `sns.barplot(x=top_teams.values, y=top_teams.index, color='steelblue')` |
| **Podium Consistency** | Filtered `positionOrder <= 3` (Top 3 finishes) | `sns.barplot(x=podiums.values, y=podiums.index, color='goldenrod')` |
| **DNF Root Causes** | Filtered `is_dnf == 1` grouped by failure status | `sns.barplot(x=dnf_causes.values, y=dnf_causes.index, color='coral')` |

---

### Key Analytical Takeaways
* **Driver Records:** **Lewis Hamilton** leads all-time wins (>100) and podiums (>200), followed by **Michael Schumacher** (91 wins, 155 podiums). Modern front-runners (**Max Verstappen**, **Fernando Alonso**) show accelerated volume due to expanded modern calendars.
* **Constructor Supremacy:** **Ferrari** leads historic totals (~250 wins), followed by **McLaren** (~185 wins), with **Mercedes** and **Red Bull Racing** overtaking legacy constructors in total modern victories.
* **Primary Failure Modes:** **Engine failures (>1,850 occurrences)** represent the largest mechanical vulnerability in F1 history, followed by on-track collisions (>2,500 total) and drivetrain sub-assembly issues (gearbox, suspension, transmission).


#  Section 2: PostgreSQL Business Logic & Relational Deep-Dive

Advanced SQL queries were executed on `fact_race_results` and `fact_lap_times` to translate telemetry data into strategic metrics: circuit conversion rates, driver racecraft indices, component failure distributions, and lap time benchmarks.

---

### Core Query Implementations & Analytical Purpose

| Query Objective | Business / Strategic Application | Core SQL Syntax & Logic |
| :--- | :--- | :--- |
| **Q1: DNF Root-Cause by Team** | Informs powertrain R&D budgeting and power-unit supplier warranty audits under the cost cap. | `WHERE is_dnf = 1 GROUP BY constructor_name, status HAVING COUNT(*) >= 3` |
| **Q2: Circuit Pole-to-Win Rate** | Guides weekend aero setup: prioritizes Saturday qualifying downforce over Sunday tire preservation for high-conversion tracks. | `SUM(CASE WHEN grid = 1 AND position_order = 1 THEN 1 ELSE 0 END)::NUMERIC / COUNT(*) * 100` |
| **Q3: Career Racecraft Delta** | Provides driver scouting departments with an objective Sunday overtaking metric isolated from car qualifying bias. | `SUM(positions_gained) AS net_positions_gained ... WHERE is_dnf = 0 HAVING COUNT(*) >= 20` |
| **Q4: Hybrid Era Points (2014+)** | Evaluates constructor point-scoring trajectories to forecast commercial revenue and sponsorship package pricing. | `SUM(points) ... WHERE race_year >= 2014 GROUP BY race_year, constructor_name` |
| **Q5 & Q6: Peak Lap Benchmarks** | Benchmarks peak aerodynamic package speed and models pit windows for fastest-lap bonus points. | `SELECT DISTINCT ON(r.race_year, r.race_name) ... ORDER BY r.race_year DESC, r.race_name, l.milliseconds ASC` |

---

### Key Analytical Takeaways
* **Component Reliability Risk:** Powertrain and transmission failures account for over 60% of team-induced mechanical retirements, making ICE and gearbox reliability the primary ROI target for cost-cap development.
* **Circuit Strategy Index:** Tracks such as the **Indian GP (100%)**, **Abu Dhabi (85%)**, and **COTA (80%)** heavily reward pole position, proving that single-lap qualifying performance outweighs race-pace setups at these venues.
* **Racecraft Benchmark:** **Fernando Alonso** leads all drivers in all-time net positions gained (**+559 places**), demonstrating elite racecraft efficiency across varied vehicle regulations.
* **Era Point Concentration:** The modern 25-point scale exponentially widened championship gaps between front-runners (**Mercedes**, **Red Bull**) and the midfield during the hybrid era.

#  Section 3: Power BI Intelligence & Multi-Era Dashboard Summary

The analytical datasets were synthesized into an interactive Power BI Executive Dashboard designed for multi-season filtering, circuit drill-throughs, and granular P1–P20 race classifications.

---

### Dashboard Architecture & Interface Layout

| Visual Component | Purpose & Metric Tracked | Key Interactive Feature |
| :--- | :--- | :--- |
| **Executive KPI Cards** | Macro context: Championship Leader, Total Leading Points, and Season DNF Total | Dynamically updates per season selection |
| **Pole-to-Win Rate Matrix** | Evaluates circuit qualifying leverage (% conversion from Grid 1 to P1) | Horizontal ranking across all historical circuits |
| **Race Craft Delta Chart** | Driver Sunday overtaking index (text{positions_gained} = {grid} - {finish}) | Leaderboard filtered by season or specific Grand Prix |
| **Constructor Standings & Podiums** | Tracks cumulative points, wins, and podium distributions | Multi-era breakdown across historical teams |
| **Official Race Classification** | Detailed race telemetry table (POS, Driver, Team, Grid, $+/-$, PTS, Status) | Granular round-by-round classification audit |

---

### Multi-Era Case Study Discoveries

* **1950 Inaugural Season Baseline:** Extreme mechanical fragility resulted in **73 DNFs across just 7 rounds**. Massive Sunday net gains (e.g., Louis Rosier $+34$) were driven primarily by surviving front-row mechanical dropouts.
* **2017 vs. 2023 United States GP (COTA):** 
  * *2017:* Lewis Hamilton converted Pole to P1, while Max Verstappen produced a $+12$ position charge ($\text{P16} \rightarrow \text{P4}$).
  * *2023:* Max Verstappen broke COTA's standard pole-to-win pattern by winning from Grid 6 ($+5$ places).
* **2024 Modern Intelligence:** 
  * *Saudi Arabian GP:* Red Bull 1-2 finish; rookie Oliver Bearman gained +4 positions ($\text{P11} \rightarrow \text{P7}$) on Ferrari debut.
  * *Dutch GP (Zandvoort):* Lando Norris converted Pole to victory ($+26\text{ PTS}$ with fastest lap), ending Verstappen's streak at Zandvoort, while Lewis Hamilton gained $+6$ places from P14. Historically at Zandvoort, Jim Clark holds the all-time victory benchmark (4 wins, 6 podiums).

---

#  Comprehensive Final Findings & Strategic Recommendations

### 1. Circuit-Driven Setup Strategies
* **Qualifying-Critical Tracks ($\ge 80\%$ Conversion):** At venues like the Indian GP ($100\%$), Abu Dhabi ($85\%$), and COTA ($80\%$), car setup and engine mapping must prioritize Saturday Q3 peak power over Sunday tire preservation.
* **Overtaking-Friendly / High-Attrition Tracks:** Setup should favor race trim and tire longevity where safety car probability and passing zones allow significant Sunday delta.

### 2. Objective Driver Scouting Framework
* Utilize the **Race Craft Delta ($+/-$)** metric alongside raw qualifying speed to evaluate drivers in midfield cars, isolating on-track overtaking racecraft from vehicle aerodynamic performance.

### 3. Engineering & R&D Prioritization Under Cost Cap
* **Powertrain Integrity:** Engine and gearbox failures account for over $60\%$ of mechanical retirements in F1 history. Under cost-cap constraints, internal combustion and hybrid durability upgrades offer higher championship ROI than minor aerodynamic package updates.

---

* ## 👤 Contact & Connect

This end-to-end pipeline and analytical report was developed by **Tejas Jagtap**. 

I am a **Data Analytics learner** focused on bridging the gap between robust data infrastructure and executive decision-making. I specialize in designing clean ETL architectures, authoring complex server-side analytical data models, and translating tracking metrics into high-impact interactive visual stories.

* **Core Competencies:** Full-Stack Data Engineering, Database Modeling (SQL/PostgreSQL), Python Automation, UI/UX Dashboard Design (Power BI).
* **📬 Let's Connect:** www.linkedin.com/in/tejasjagtap01 
* **GitHub:** https://github.com/tejasjagtap-01
* Portfolio/Website:  https://mavenshowcase.com/profile/788113c0-5061-70b8-3303-fdd74d72b9be 

## 📜 License
This project is licensed under the MIT License - see the LICENSE file for details.


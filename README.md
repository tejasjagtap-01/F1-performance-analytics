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

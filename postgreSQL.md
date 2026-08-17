# The Master Table: Race Results 
```
CREATE TABLE fact_race_results (
    result_id INT,
    race_id INT,
    driver_id INT,
    constructor_id INT,
    number VARCHAR(50),
    grid INT,
    position VARCHAR(50),
    position_text VARCHAR(50),
    position_order INT,
    points NUMERIC(10,2),
    laps INT,
    race_time VARCHAR(100),
    milliseconds VARCHAR(100),
    fastest_lap VARCHAR(50),
    rank_order VARCHAR(50),
    fastest_lap_time VARCHAR(50),
    fastest_lap_speed VARCHAR(50),
    status_id INT,
    race_year INT,
    race_round INT,
    circuit_id INT,
    race_name VARCHAR(150),
    driver_ref VARCHAR(100),
    forename VARCHAR(100),
    surname VARCHAR(100),
    nationality VARCHAR(100),
    constructor_ref VARCHAR(100),
    constructor_name VARCHAR(100),
    positions_gained INT,
    driver_full_name VARCHAR(150),
    is_dnf INT,
    status VARCHAR(150)
);
```

## Fact Table 1. Pit Stops 
```
CREATE TABLE fact_pit_stops(
	race_id INT,
	driver_id INT,
	stop_number INT,
	lap INT,
	pit_time VARCHAR(50),
	duration_str VARCHAR(50),
	duration_sec NUMERIC
)
```

## Fact Table 2. Lap Times
```
CREATE TABLE fact_lap_times(
	race_id INT,
	driver_id INT,
	lap INT,
	position INT,
	lap_time_str VARCHAR(20),
	milliseconds INT
)
```

---


# SQL Exploration & Business Queries 


###  Q1. Which teams suffer from the most retirements (DNFs), and what are the specific reasons (Engine, Gearbox, Accident, etc.) causing them?
```
SELECT 
	constructor_name,
	status AS dnf_reason,
	COUNT(*) AS total_occurrences 
FROM fact_race_results
WHERE is_dnf = 1
	AND status NOT IN ('Finished','+1 Lap', '+2 Laps', '+3 Laps')
GROUP BY constructor_name, status
HAVING COUNT(*) >= 3
ORDER BY constructor_name, total_occurrences DESC
```

```
What it finds:
Car breaks down because of an engine failure, that's an engineering problem.
If your driver keeps hitting barriers, that's a driver problem.
A team principal needs to know what is causing the majority of their DNFs.
```
----

### Q2. At which circuits does starting on Pole Position (Grid Position 1) most frequently result in winning the race?
```
SELECT 
	race_name,
	COUNT(*) AS total_races_analyzed,
	SUM(CASE WHEN grid = 1 AND position_order = 1 THEN 1 ElSE 0 END) AS pole_to_win_count,
	ROUND(
		(SUM(CASE WHEN grid = 1 AND position_order = 1 THEN 1 ELSE 0 END)::NUMERIC / COUNT(*)) * 100,1
	)AS pole_win_conversion_pct 
FROM fact_race_results
WHERE grid = 1 AND is_dnf = 0
GROUP BY race_name
HAVING COUNT(*) >= 5
ORDER BY pole_win_conversion_pct DESC
```

```
What it finds:

Measures the historical probability of converting Pole Position (Grid 1) into an outright race victory across different Grand Prix circuits
```

--- 

### Q3. Which drivers have gained the highest total number of places on track relative to where they started on the grid?
```
SELECT 
	driver_full_name,
	COUNT(*) AS races_started,
	SUM(positions_gained) AS net_positions_gained,
	ROUND(AVG(positions_gained),1) AS avg_gained_per_race
FROM fact_race_results
WHERE is_dnf = 0
GROUP BY driver_full_name
HAVING COUNT(*) >= 20
ORDER BY net_positions_gained DESC
LIMIT 10
```

```
What it finds:

Ranks the drivers who gain the highest net positions from start to finish across their careers, highlighting racecraft efficiency.
```

---

### Q4. How many total Constructors' Championship points has each team scored per season in the modern Hybrid Era (2014+)?
```
SELECT
	race_year,
	constructor_name,
	SUM(points) AS total_season_points,
	COUNT(DISTINCT race_id) AS total_races
FROM fact_race_results
WHERE race_year >= 2014
GROUP BY race_year, constructor_name
HAVING SUM(points) > 0
ORDER BY race_year DESC, total_season_points DESC
```

```
What it finds:

Aggregates total Constructors' Championship points scored by each team season-by-season throughout the V6 Turbo-Hybrid regulatory era (2014 onward).
```

---

### Q5. What is the single fastest lap recorded of the lap times dataset for race track, and who set it?
```
SELECT
	r.race_year,
	r.race_name,
	r.driver_full_name AS fastest_driver,
	r.constructor_name AS fastest_team,
	ROUND(MIN(l.milliseconds) / 1000.0, 3) AS fastest_lap_seconds
FROM fact_lap_times l
JOIN fact_race_results r
ON l.race_id = r.race_id AND l.driver_id = r.driver_id
WHERE r.race_year >= 2018
GROUP BY r.race_year, r.race_name, r.driver_full_name, r.constructor_name
ORDER BY r.race_year DESC, r.race_name
```

```
What it finds:

Pinpoints the absolute fastest single race lap recorded at each circuit per season (from 2018 onward), linking the lap time directly to the driver and car that achieved it.
```

---

### Q6. What is the single fastest lap recorded in the lap times dataset for each race track, and who set it?
```
SELECT DISTINCT ON(r.race_year, r.race_year)
	r.race_year,
	r.race_name,
	r.driver_full_name AS fastest_driver,
	r.constructor_name AS fastest_team,
	ROUND(l.milliseconds / 1000.0,3) AS fastest_lap_seconds
FROM fact_lap_times l
JOIN fact_race_results r
ON l.race_id = r.race_id AND l.driver_id = r.driver_id
WHERE r.race_year >= 2018 
	AND l.milliseconds IS NOT NULL 
	AND l.milliseconds > 0
ORDER BY r.race_year DESC, r.race_name, l.milliseconds ASC
```

```
What it finds:

Pinpoints the absolute fastest single race lap recorded at each circuit per season (from 2018 onward), linking the lap time directly to the driver and car that achieved it.
```

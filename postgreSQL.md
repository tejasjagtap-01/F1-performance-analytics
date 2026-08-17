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

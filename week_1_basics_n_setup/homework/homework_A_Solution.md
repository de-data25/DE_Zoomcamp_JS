## Week 1 Homework

In this homework we'll prepare the environment 
and practice with Docker and SQL


## Question 1. Knowing docker tags

Run the command to get information on Docker 

```docker --help```

Now run the command to get help on the "docker build" command

Which tag has the following text? - *Write the image ID to the file* 

Answer:

- `--iidfile string`


## Question 2. Understanding docker first run 

Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash.
Now check the python modules that are installed ( use pip list). 
How many python packages/modules are installed?

Commands:
    docker run -it --entrypoint=bash python:3.9
    pip list

Answer:
    - 3


# Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from January 2019:

```wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz```

You will also need the dataset with zones:

```wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv```

Download this data and put it into Postgres (with jupyter notebooks or with a pipeline)

Notes: URL "https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv" wasn't available for download so I downloaded from "https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv "

Also, had to use "python -m wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz" for .gz file, instead of "wget".


# bash 1
  winpty docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v //c/DE_Zoomcamp/data-engineering-zoomcamp-main/week_1_basics_n_setup/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  --network=pg-network \
  --name pgdatabase \
  postgres:13

# bash 2
jupyter notebook

to load datasets I run code stored in scripts:

- upload-data-green.ipynb
- upload-data-zone-lookup.ipynb


## Question 3. Count records 

How many taxi trips were totally made on January 15?

Tip: started and finished on 2019-01-15. 

Remember that `lpep_pickup_datetime` and `lpep_dropoff_datetime` columns are in the format timestamp (date and hour+min+sec) and not in date.

SQL code:

SELECT COUNT(*)
FROM PUBLIC.GREEN_TAXI_DATA
WHERE DATE(LPEP_PICKUP_DATETIME) = '2019-01-15'
	AND DATE(LPEP_DROPOFF_DATETIME) = '2019-01-15';

Answer:

- 20530

## Question 4. Largest trip for each day

Which was the day with the largest trip distance
Use the pick up time for your calculations.

SQL Code:

SELECT MAX(TRIP_DISTANCE),
	DATE(LPEP_PICKUP_DATETIME) AS DATE_PICKUP
FROM PUBLIC.GREEN_TAXI_DATA
WHERE DATE(LPEP_PICKUP_DATETIME) in ('2019-01-10','2019-01-15','2019-01-18','2019-01-28')
GROUP BY DATE_PICKUP;

Answer:

- 2019-01-15  (117.99)


## Question 5. The number of passengers

In 2019-01-01 how many trips had 2 and 3 passengers?
 
SQL Code:

SELECT COUNT(*),
	PASSENGER_COUNT
FROM PUBLIC.GREEN_TAXI_DATA
WHERE PASSENGER_COUNT in ('2','3')
	AND DATE(LPEP_PICKUP_DATETIME) = '2019-01-01' --and date(lpep_dropoff_datetime) = '2019-01-01'
GROUP BY PASSENGER_COUNT;

Answer:

- 2: 1282 ; 3: 254

## Question 6. Largest tip

For the passengers picked up in the Astoria Zone which was the drop off zone that had the largest tip?
We want the name of the zone, not the id.

Note: it's not a typo, it's `tip` , not `trip`

SQL Code:

SELECT MAX(A.TIP_AMOUNT),
	A.DOLOCATION_ZONE
FROM
	(SELECT GTD.TIP_AMOUNT,
			TXLP."Zone" AS PULOCATION_ZONE,
			TXLD."Zone" AS DOLOCATION_ZONE
		FROM PUBLIC.GREEN_TAXI_DATA AS GTD
		LEFT JOIN PUBLIC.TAXI_ZONE_LOOKUP AS TXLP ON GTD."PULocationID" = TXLP."LocationID"
		LEFT JOIN PUBLIC.TAXI_ZONE_LOOKUP AS TXLD ON GTD."DOLocationID" = TXLD."LocationID"
		WHERE TXLP."Zone" like 'Astoria%') A
GROUP BY A.DOLOCATION_ZONE
ORDER BY MAX(A.TIP_AMOUNT) DESC
LIMIT 1;

Answer:

- Long Island City/Queens Plaza (88.0)


## Submitting the solutions

* Form for submitting: [form](https://forms.gle/EjphSkR1b3nsdojv7)
* You can submit your homework multiple times. In this case, only the last submission will be used. 

Deadline: 30 January (Monday), 22:00 CET


## Solution

We will publish the solution here

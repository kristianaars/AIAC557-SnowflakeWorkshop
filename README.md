This workshop covers the most important features of Snowflake. We will learn how to create and manipulate databases, as well as how to import large datasets. At the end, we will perform a statistical analysis using real life data.

# 0. Download demo file
Download the demo data from here: [Download the Citibike Ridefile](https://1drv.ms/u/s!AglMzqUexDSxjZteYWBitXwT11SENw?e=KeF8Df)

# 1. Connect to database

### Terminal
If using Snowflake SQL client you may connect like so:

```bash
snowsql -a <ACCOUNT-NAME> -u <USERNAME>
```

# 2. Basic Queries

Create a database called `my-sf-db`
```sql
CREATE DATABASE my_sf_db;
```

Make a new table `users` in the database
```sql
CREATE TABLE users (
	id NUMBER AUTOINCREMENT,
	first_name STRING,
	surname STRING,
	email STRING,
	born_year NUMBER
)
```

Insert some data into your new table like so
```sql
INSERT INTO users (first_name, surname, email, born_year) VALUES
(
	'<YOUR_FIRSTNAME>',
	'<YOUR_SURNAME>',
	'<YOUR_EMAIL>',
	1998
),
(
	'Barack',
	'Obama',
	'barack.obama@yahoo.com',
	1961
),
(
	'Milan',
	'Newar',
	'milan.newar@gmail.com',
	1968
);
```

View all the users
```sql
SELECT * FROM users;
```

Select all users, and calculate their age
```sql
SELECT first_name, surname, email, 2023 - born_year as age FROM users;
```

Only select users where the age is grated than 60
```sql
SELECT first_name, surname, email, 2023 - born_year as age FROM users
WHERE age < 60;
```

Only select users with a gmail registered
```sql
SELECT email FROM users
WHERE email LIKE '%gmail.com';
```

# 3. Import Data

Use the SnowSQL
## 3.1 From marketplace

1. Go to Snowflake marketplace
2. Search for weather-data
3. Download 

## 3.2 From file

First, lets create the database and the table
```sql
CREATE DATABASE citibike;
```
```sql
CREATE TABLE rides (
	ride_id STRING,
	rideable_type STRING,
	started_at DATETIME,
	ended_at DATETIME,
	start_station_name STRING,
	start_station_id STRING,
	end_station_name STRING,
	end_station_id STRING,
	start_lat NUMBER,
	start_lng NUMBER,
	end_lat NUMBER,
	end_lng NUMBER,
	member_casual STRING
)
```

Now, lets upload the file for staging
```sql
PUT file://<PATH-TO-RIDE-DATA-CSV> @citibike.public.%rides;
```

After staging is complete, we can insert the staged data into our new table
```sql
COPY INTO rides
  FROM @%rides
  FILE_FORMAT = (type = csv field_optionally_enclosed_by='"')
  ON_ERROR = 'skip_file';
```

Now, lets try to join the weather data imported from the marketplace, with the different rides.
```sql
SELECT * FROM CITIBIKE.PUBLIC.rides citibike_data
INNER JOIN GLOBAL_WEATHER__CLIMATE_DATA_FOR_BI.STANDARD_TILE.HISTORY_DAY weather_data
ON weather_data.date_valid_std = TO_DATE(citibike_data.started_at)
WHERE weather_data.postal_code LIKE '10257'
```

That took some time... Let's try upgrading to a larger **Virtual Warehouse**
```sql
CREATE OR REPLACE WAREHOUSE my-xl-warehouse WITH WAREHOUSE_SIZE='X-LARGE'
```

```sql
# Lets try a bigger warehouse
USE WAREHOUSE my-xl-warehouse
```

```sql
SELECT * FROM CITIBIKE.PUBLIC.rides citibike_data
INNER JOIN GLOBAL_WEATHER__CLIMATE_DATA_FOR_BI.STANDARD_TILE.HISTORY_DAY weather_data
ON weather_data.date_valid_std = TO_DATE(citibike_data.started_at)
WHERE weather_data.postal_code LIKE '10257'
```

# 4. Python Connector
Continue in [notebook.ipynb](./notebook.ipynb)

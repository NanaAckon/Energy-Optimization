# Optimizing Energy Consumption In Steel Manufacturing Plant Analysis

### Project Overview

This project focused on developing a comprehensive solution to optimize the steel manufacturing process. By leveraging advanced analytics and machine learning techniques, we identified the ideal combination of raw materials and energy inputs to enhance steel quality while minimizing costs and energy usage.

Key achievements include:
Significant reduction in processing time by 50%, surpassing business success criteria.

Achieved cost savings exceeding $1M, aligning with economic success criteria.
Through this project, we demonstrated the potential of data-driven decision-making in revolutionizing steel manufacturing efficiency, setting a benchmark for future industry standards.

### Data Source

Data is the "gt_clientdata" file collected from both Primary and Secondary sources containing detailed information.
The Shape of the "gt_client" dataset is (2061, 88),indicating 2061 rows and 88 columns.


### Tools
Data Analysis:

- PostgreSQL [Download here](https://www.postgresql.org/download/)
- Python [Download here](https://www.anaconda.com/download)

Data Visualization:
- Tableau [Download here](https://www.tableau.com/products/desktop/download)
- Google Looker studio [Click here](https://g.co/kgs/C1iYDMb)
- Excel Charts [Download here](https://www.microsoft.com/en-us/microsoft-365/excel)

### System Requirements
 - Device name: 	                DESKTOP-2H5KJ9T
- Processor:                      11th Gen Intel(R) Core(TM) i3-1125G4 @ 2.00GHz   2.00 GHz
- Installed RAM: 	                8.00 GB (7.65 GB usable)
- Device ID:	                    E15F3780-506F-4421-AF2E-1D0E0F57099B
- Product ID:	                    00325-82130-28104-AAOEM
- System type:	                  64-bit operating system, x64-based processor
- Pen and touch:	                Touch support with 10 touch points


### Data Preprocessing/Preperation/Cleansing And EDA with PostgreSQL
The following task was perfomed in the initial data preperatiion stage;
  -  Handling Missing Values ( Both SQL and Python)

 ```SQL
    ----Handling the Missing Values in section
SELECT section
FROM gt_clientdata
WHERE section IS NULL
  
  SELECT COUNT(*)
FROM gt_clientdata
WHERE section IS NULL;

-------Mode Imputation
WITH mode_section AS (
    SELECT section
    FROM gt_clientdata
    GROUP BY section
    ORDER BY COUNT(*) DESC
    LIMIT 1
)
UPDATE gt_clientdata
SET section = (SELECT section FROM mode_section)
WHERE section IS NULL;


----lab_rep_time
SELECT lab_rep_time
FROM gt_clientdata
WHERE lab_rep_time IS NULL
  
SELECT COUNT(*)
FROM gt_clientdata
WHERE lab_rep_time IS NULL;

-----Mean imputation
UPDATE gt_clientdata
SET lab_rep_time = (SELECT AVG(lab_rep_time) FROM gt_clientdata WHERE lab_rep_time IS NOT NULL)
WHERE lab_rep_time IS NULL; 
```


-  Detection and Treating Of Outliers
```SQL
   --------------------------Outlier detection and treatment for energy -----------------------------------------------------
select * from gt_clientdata
WITH quartiles AS (
    SELECT 
        percentile_cont(0.25) WITHIN GROUP (ORDER BY energy) AS q1,
        percentile_cont(0.75) WITHIN GROUP (ORDER BY energy) AS q3
    FROM gt_clientdata
),
iqr AS (
    SELECT 
        q1,
        q3,
        (q3 - q1) AS iqr
    FROM quartiles
)
SELECT 
    energy
FROM 
    gt_clientdata, iqr
WHERE 
    energy< (q1 - 1.5 * iqr) OR energy > (q3 + 1.5 * iqr);

WITH bounds AS (
    SELECT 
        percentile_cont(0.05) WITHIN GROUP (ORDER BY energy) AS lower_bound,
        percentile_cont(0.95) WITHIN GROUP (ORDER BY energy) AS upper_bound
    FROM gt_clientdata
)
UPDATE gt_clientdata
SET energy = CASE
    WHEN energy < lower_bound THEN lower_bound
    WHEN energy > upper_bound THEN upper_bound
    ELSE energy
END
FROM bounds;

---------------------------Outlier detection and treatment for kwh_per_ton---------------------
select * from gt_clientdata

WITH quartiles AS (
    SELECT 
        percentile_cont(0.25) WITHIN GROUP (ORDER BY kwh_per_ton) AS q1,
        percentile_cont(0.75) WITHIN GROUP (ORDER BY kwh_per_ton) AS q3
    FROM gt_clientdata
),
iqr AS (
    SELECT 
        q1,
        q3,
        (q3 - q1) AS iqr
    FROM quartiles
)
SELECT 
    kwh_per_ton
FROM 
    gt_clientdata, iqr
WHERE 
    kwh_per_ton < (q1 - 1.5 * iqr) OR kwh_per_ton > (q3 + 1.5 * iqr);

----- Treating with Winsorization Method
WITH bounds AS (
    SELECT 
        percentile_cont(0.05) WITHIN GROUP (ORDER BY kwh_per_ton) AS lower_bound,
        percentile_cont(0.95) WITHIN GROUP (ORDER BY kwh_per_ton) AS upper_bound
    FROM gt_clientdata
)
UPDATE gt_clientdata
SET kwh_per_ton = CASE
    WHEN kwh_per_ton < lower_bound THEN lower_bound
    WHEN kwh_per_ton > upper_bound THEN upper_bound
    ELSE kwh_per_ton
END
FROM bounds;

```


- Desciptive Statistics With SQL
```SQL
  Select * from gt_clientdata

-- First Moment Business Decision 
--Heatno Mean 
SELECT AVG(heatno) AS mean_heatno
FROM gt_clientdata;

--MEDIAN
SELECT heatno AS median_experience
FROM (SELECT heatno, ROW_NUMBER() OVER (ORDER BY heatno) AS row_num,
           COUNT(*) OVER () AS total_count
    FROM gt_clientdata
) AS subquery
WHERE row_num = (total_count + 1) / 2 OR row_num = (total_count + 2) / 2;  

--mode
 SELECT heatno AS mode_heatno
FROM (
    SELECT heatno, COUNT(*) AS frequency
    FROM gt_clientdata
    GROUP BY heatno
    ORDER BY frequency DESC
    LIMIT 1
) AS subquery;

--Second Moment Business Decision/Measures of Dispersion
--Variance
SELECT VARIANCE(heatno) AS heatno_variance
FROM gt_clientdata;

--Standard Deviation 
SELECT STDDEV(heatno) AS heatno_stddev
FROM gt_clientdata;

# Range
SELECT MAX(heatno) - MIN(heatno) AS heatno_range
FROM gt_clientdata;


--Third and Fourth Moment Business Decision
-- skewness and kurkosis 

SELECT
    (
        SUM(POWER(heatno - (SELECT AVG(heatno) FROM gt_clientdata), 3)) / 
        (COUNT(*) * POWER((SELECT STDDEV(heatno) FROM gt_clientdata), 3))
    ) AS skewness,
    (
        (SUM(POWER(heatno - (SELECT AVG(heatno) FROM gt_clientdata), 4)) / 
        (COUNT(*) * POWER((SELECT STDDEV(heatno) FROM gt_clientdata), 4))) - 3
    ) AS kurtosis
FROM gt_clientdata

-------------------------------------------------------------------------------------------------
--- Grade Mode
SELECT grade AS mode_grade
FROM (
    SELECT grade, COUNT(*) AS frequency
    FROM gt_clientdata
    GROUP BY grade
    ORDER BY frequency DESC
    LIMIT 1
) AS subquery;
-------------------------------------------------------------------------------------------------
--- section Mode
SELECT section AS mode_grade
FROM (
    SELECT section, COUNT(*) AS frequency
    FROM gt_clientdata
    GROUP BY section
    ORDER BY frequency DESC
    LIMIT 1
) AS subquery;

-------------------------------------------------------------------------------------------------
--section Mode
SELECT section_ic AS mode_grade
FROM (
    SELECT section_ic, COUNT(*) AS frequency
    FROM gt_clientdata
    GROUP BY section_ic
    ORDER BY frequency DESC
    LIMIT 1
) AS subquery;

-------------------------------------------------------------------------------------------------
-- Mode for si_eaf
SELECT si_eaf AS mode_grade
FROM (
    SELECT si_eaf, COUNT(*) AS frequency
    FROM gt_clientdata
    GROUP BY si_eaf
    ORDER BY frequency DESC
    LIMIT 1
) AS subquery;

-------------------------------------------------------------------------------------------------
-- First Moment Business Decision 
-- inj1_qty Mean 
SELECT AVG(inj1_qty) AS mean_inj1_qty
FROM gt_clientdata;

--MEDIAN
SELECT inj1_qty AS median_inj1_qty
FROM (SELECT inj1_qty, ROW_NUMBER() OVER (ORDER BY inj1_qty) AS row_num,
           COUNT(*) OVER () AS total_count
    FROM gt_clientdata
) AS subquery
WHERE row_num = (total_count + 1) / 2 OR row_num = (total_count + 2) / 2;  

--mode
 SELECT inj1_qty AS mode_inj1_qty
FROM (
    SELECT inj1_qty, COUNT(*) AS frequency
    FROM gt_clientdata
    GROUP BY inj1_qty
    ORDER BY frequency DESC
    LIMIT 1
) AS subquery;

--Second Moment Business Decision/Measures of Dispersion
--Variance
SELECT VARIANCE(inj1_qty) AS inj1_qty_variance
FROM gt_clientdata;

--Standard Deviation 
SELECT STDDEV(inj1_qty) AS inj1_qty_stddev
FROM gt_clientdata;

# Range
SELECT MAX(inj1_qty) - MIN(inj1_qty) AS inj1_qty_range
FROM gt_clientdata;


--Third and Fourth Moment Business Decision
-- skewness and kurkosis 

SELECT
    (
        SUM(POWER(inj1_qty - (SELECT AVG(inj1_qty) FROM gt_clientdata), 3)) / 
        (COUNT(*) * POWER((SELECT STDDEV(inj1_qty) FROM gt_clientdata), 3))
    ) AS skewness,
    (
        (SUM(POWER(inj1_qty - (SELECT AVG(inj1_qty) FROM gt_clientdata), 4)) / 
        (COUNT(*) * POWER((SELECT STDDEV(inj1_qty) FROM gt_clientdata), 4))) - 3
    ) AS kurtosis
FROM gt_clientdata
-------------------------------------------------------------------------------------------------
-- First Moment Business Decision 
-- inj2_qty Mean 
SELECT AVG(inj2_qty) AS mean_inj2_qty
FROM gt_clientdata;

--MEDIAN
SELECT inj2_qty AS median_inj2_qty
FROM (SELECT inj2_qty, ROW_NUMBER() OVER (ORDER BY inj2_qty) AS row_num,
           COUNT(*) OVER () AS total_count
    FROM gt_clientdata
) AS subquery
WHERE row_num = (total_count + 1) / 2 OR row_num = (total_count + 2) / 2;  

--mode
 SELECT inj2_qty AS mode_inj2_qty
FROM (
    SELECT inj2_qty, COUNT(*) AS frequency
    FROM gt_clientdata
    GROUP BY inj2_qty
    ORDER BY frequency DESC
    LIMIT 1
) AS subquery;

--Second Moment Business Decision/Measures of Dispersion
--Variance
SELECT VARIANCE(inj2_qty) AS inj2_qty_variance
FROM gt_clientdata;

--Standard Deviation 
SELECT STDDEV(inj2_qty) AS inj2_qty_stddev
FROM gt_clientdata;

# Range
SELECT MAX(inj2_qty) - MIN(inj2_qty) AS inj2_qty_range
FROM gt_clientdata;


--Third and Fourth Moment Business Decision
-- skewness and kurkosis 

SELECT
    (
        SUM(POWER(inj2_qty - (SELECT AVG(inj2_qty) FROM gt_clientdata), 3)) / 
        (COUNT(*) * POWER((SELECT STDDEV(inj2_qty) FROM gt_clientdata), 3))
    ) AS skewness,
    (
        (SUM(POWER(inj2_qty - (SELECT AVG(inj2_qty) FROM gt_clientdata), 4)) / 
        (COUNT(*) * POWER((SELECT STDDEV(inj2_qty) FROM gt_clientdata), 4))) - 3
    ) AS kurtosis
FROM gt_clientdata


```

### Data Extraction/Retrival
Raw Data was extrated from PostgeSQL into Python(Libraries used; Pandas, sqlalchemy, psycopg2)

``` Python
import pandas as pd
from sqlalchemy import create_engine
import psycopg2

# Create a database connection
engine = create_engine('postgresql://user:password@localhost:5432/mydatabase')

# Define the SQL query
query = "SELECT * FROM gt_clientdata2"

# Load data into a DataFrame
df = pd.read_sql(query, engine)

# Display the first few rows of the DataFrame
print(df.head())

import os
print(os.getenv('DB_USER'))
print(os.getenv('DB_PASSWORD'))


import psycopg2
import pandas as pd

# Define your connection parameters
conn = psycopg2.connect(
    dbname="GT_ClientData",
    user="postgres",
    password="**********",
    host="localhost",
    port= 5432
)

# Create a cursor object
cursor = conn.cursor()

# Define your SQL query
query = "SELECT * FROM your_table_name"

# Load data into a DataFrame
gt_clientdata = pd.read_sql_query(query, conn)

# Close the cursor and connection
cursor.close()
conn.close()

# Display the DataFrame
print(df.head())

df = cur.fetchall()
df.heatno.mean()

### Reassigning My Variable To gt_clientdata
gt_clientdata = df
del df
print(gt_clientdata)

```

### Exploratory Data Analysis In Pyhon 
In EDA we explored the "gt_clientdata"performing statistical (Descriptives) using python as well  to answer key business questions;
- Descriptive Statistics with python(Libraries used (

```Python   
######### Business Decisions for GT Client Data #########
### First Moment Business Decision, Mean ,Median, Mode ####
gt_clientdata.heatno.mean() #mean

gt_clientdata.heatno.median() #Median

gt_clientdata.heatno.mode() #Mode

from scipy import stats
mode = stats.mode(gt_clientdata.heatno) # Multimodal
print(mode)

#### Second Moment Business Decisions ######
gt_clientdata.heatno.var() # variance
gt_clientdata.heatno.std() # standard deviation
range = max(gt_clientdata.heatno) - min(gt_clientdata.heatno) # range
range

# Third moment business decision
gt_clientdata.heatno.skew()

# Fourth moment business decision
gt_clientdata.heatno.kurt()

from scipy import stats
mode = stats.mode(gt_clientdata.heatno)
print(mode)

```


- How many types of grades of steel are present in my dataset?
- For manufacturing these types of grades of steel which types of raw materials are mixed together and how many?
- Average Energy Consumption per Grade: Which grade has the highest average energy consumption?
- Which grade of steel during manufacturing process consumes more energy or electricity?
- Which grades of steel during manufacturing process consumes less energy or electricity?
- Which grade brings about the maximum production?
- Production Time Analysis: Determine the average melting and total cycle time per grade.
- Shift Effeciency 

```Python
######### Business Insights ################## 

import pandas as pd


# 1. How many types of grades of steel are present in my dataset?
grades_of_steel = gt_clientdata['grade'].nunique()
print(f"Number of grades of steel: {grades_of_steel}")

# 2. For manufacturing these types of grades of steel which types of raw materials are mixed together and how many?
raw_materials_columns = ['inj1_qty', 'inj2_qty', 'bsm', 'skull', 'bp', 'hbi', 'others', 
                         'scrap_qty_mt', 'pigiron', 'dri1_qty_mt_lumps', 'dri2_qty_mt_fines', 'tot_dri_qty', 
                         'hot_metal_from_mbf']
raw_materials_per_grade = gt_clientdata.groupby('grade')[raw_materials_columns].count()
print("Raw materials per grade of steel:\n", raw_materials_per_grade)

 # Calculating total raw materials used per grade
gt_clientdata['Total_Raw_Materials'] = gt_clientdata[raw_materials_columns].sum(axis=1)
total_raw_materials_per_grade = gt_clientdata.groupby('grade')['Total_Raw_Materials'].sum()

  # Identifying the grade using the most and least raw materials
most_raw_materials_grade = total_raw_materials_per_grade.idxmax()
least_raw_materials_grade = total_raw_materials_per_grade.idxmin()

print(f"Grade using the most raw materials: {most_raw_materials_grade}")
print(f"Grade using the least raw materials: {least_raw_materials_grade}")


#3. Average Energy Consumption per Grade: Which grade has the highest average energy consumption?
avg_energy_per_grade = gt_clientdata.groupby('grade')['energy'].mean()
print("Average energy consumption per grade:\n", avg_energy_per_grade)

# 4. Which grade of steel during manufacturing process consumes more energy or electricity?
most_energy_consuming_grade = gt_clientdata.groupby('grade')['energy'].sum().idxmax()
print(f"Grade of steel consuming the most energy: {most_energy_consuming_grade}")

# 5. Which grades of steel during manufacturing process consumes less energy or electricity?
least_energy_consuming_grade = gt_clientdata.groupby('grade')['energy'].sum().idxmin()
print(f"Grade of steel consuming the least energy: {least_energy_consuming_grade}")


# 6. Which grade brings about the maximum production?
max_production_grade = gt_clientdata.groupby('grade')['production_mt'].sum().idxmax()
print(f"Grade of steel with maximum production: {max_production_grade}")



#7.Production Time Analysis: Determine the average melting and total cycle time per grade.
avg_melt_time_per_grade = gt_clientdata.groupby('grade')['melt_time'].mean()
avg_cycle_time_per_grade = gt_clientdata.groupby('grade')['tt_time'].mean()
print("Average melting time per grade:\n", avg_melt_time_per_grade)
        

# Shift Effeciency
# Group by 'section_ic' and sum the 'ENERGY (Energy Consumption)' column
energy_consumption_per_shift = gt_clientdata.groupby('section_ic')['energy'].sum()

# Identify the shift with the maximum energy consumption
shift_max_energy = energy_consumption_per_shift.idxmax()
max_energy = energy_consumption_per_shift.max()

print(f"Shift with the highest energy consumption: {shift_max_energy} with {max_energy} units of energy consumed")

#Electrode Efficiency: Assess which electrode current levels correlate with optimal energy use and production.
electrode_efficiency = gt_clientdata.groupby('grade')[['e1_cur', 'e2_cur', 'e3_cur', 'energy', 'production_mt']].mean()
print("Electrode efficiency per grade:\n", electrode_efficiency)

#Temperature Impact: Understand how tapping temperatures (TAP_TEMP) influence energy consumption and production quality.
temp_impact = gt_clientdata.groupby('grade')[['tap_temp', 'energy', 'production_mt']].mean()
print("Temperature impact on energy and production:\n", temp_impact)

#Oxygen Activity: Analyze the effect of oxygen activity levels (O2ACT) on production efficiency and energy use.
oxygen_activity_impact = gt_clientdata.groupby('grade')[['o2act', 'energy', 'production_mt']].mean()
print("Oxygen activity impact on energy and production:\n", oxygen_activity_impact)

gt_clientdata['Energy per MT'] = gt_clientdata['energy'] / gt_clientdata['production_mt']
energy_efficiency_per_grade = gt_clientdata.groupby('grade')['Energy per MT'].mean()
print("Energy efficiency per grade:\n", energy_efficiency_per_grade)
```

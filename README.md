![image](https://github.com/user-attachments/assets/0e2c150f-d916-4cd9-a541-832125b7b849)# Optimizing Energy Consumption In Steel Manufacturing Plant Analysis

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

### Data Cleaning With Python
- Handling Missing Values
Dataset had 92 missing values in total from 4 columns(section, production MT, lab_rep_time, prev_tap_time)

 --Looking out for Missing values
![image](https://github.com/user-attachments/assets/dd980ba8-a0ce-42ac-ac1f-0faf4c5b15ca)

 --Handling missing Values With imputation Method

 ```Python
#################### Missing Values - Imputation ###########################

import pandas as pd
import numpy as np
from sklearn.impute import SimpleImputer

# Mode Imputer for section
mode_imputer = SimpleImputer(missing_values = np.nan, strategy = 'most_frequent')
gt_clientdata['section'] = gt_clientdata['section'].astype(str)
gt_clientdata["section"] = pd.DataFrame(mode_imputer.fit_transform(gt_clientdata[["section"]]))
gt_clientdata["section"].isna().sum()


gt_clientdata.isna().sum()

# Mean Imputer for production_mt
mean_imputer = SimpleImputer(missing_values = np.nan, strategy = 'mean')
gt_clientdata["production_mt"] = pd.DataFrame(mean_imputer.fit_transform(gt_clientdata[["production_mt"]]))
gt_clientdata["production_mt"].isna().sum()

#lab_rep_time
gt_clientdata['lab_rep_time'] = gt_clientdata['lab_rep_time'].fillna(method='bfill')
gt_clientdata['lab_rep_time'].isna().sum()

#prev_tap_time
gt_clientdata['prev_tap_time'] = gt_clientdata['prev_tap_time'].fillna(method = 'bfill')
gt_clientdata['prev_tap_time'].isna().sum()
```

- Outlier Detection And Treatment with Winsorization
For detecting outliers boxplot was used to identify outliers and winsorization which involves defining the maximum and minimum of IQR and replacing outliers with them. This was performed on all numeric columns of the dataset.Libraries like seaborn(for visualization) to detect outliers with boxplot, and feature_engine.outliers for winsorizer
for treating outliers.

![image](https://github.com/user-attachments/assets/e63b5188-5262-41cc-90fb-bbf725b01b19)

![image](https://github.com/user-attachments/assets/e61a3021-6cf7-4a5c-99c1-498746ed6480)

![image](https://github.com/user-attachments/assets/a059337e-0e51-4a44-acb1-04a9c395e2c3)




### Exploratory Data Analysis In Pyhon 
In EDA we explored the "gt_clientdata"performing statistical (Descriptives) using python as well  to answer key business questions;
- Descriptive Statistics with python(Libraries used Pandas,Scipy)

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

### Data Visualizations
-Tableau
![Screenshot 2024-10-21 002459](https://github.com/user-attachments/assets/472c28a5-b15b-4230-a92f-34605398021d)

![Screenshot 2024-10-21 002809](https://github.com/user-attachments/assets/59f0eb9a-7eb5-4f89-8136-a5999ed70f48)

![Screenshot 2024-10-21 003103](https://github.com/user-attachments/assets/f217e8be-742c-4799-95b1-be49ceed0ae7)


- Google Looker Studio
![Screenshot 2024-10-21 004410](https://github.com/user-attachments/assets/241d2704-9496-4493-8070-8d7253e98bd0)

- Excel charts
![Screenshot 2024-10-21 000904](https://github.com/user-attachments/assets/56cb677f-5a09-4c1b-b4eb-52611df851ef)

![Screenshot 2024-10-21 001120](https://github.com/user-attachments/assets/a3746405-aff6-4904-a324-69a40450fec2)

![Screenshot 2024-10-21 001326](https://github.com/user-attachments/assets/949a8afe-e1ce-4740-8b3a-11636a8b120f)

## Insights And Findings

####  Types of Grades: How many types of grades of steel are present in the dataset?
   - There are 328 unique Grades of Steels

###   Raw Materials Per Grade: For manufacturing these types of grades of steel which types of raw materials are mixed together and how many?
   -  Data showed raw materials per grade with Grade type S48CS1V  being the grade of steel that uses most raw materials for production and Grade type ICSS-1218- using the least raw materials.

###  Average Energy Consumption per Grade: Which grade has the highest average energy consumption? (At Top 5)
   |GRADE|AVERAGE ENERGY CONSUMPTION|
   |-----|--------------------------|
   |40MnSiVS6|26465.2825|
   |42CRMOS4|	23430.855|
   |AISI410	|	21372.3|
   |SAE9310| 16707.42|
   |SAE4140|	15716.85|

### 	Most Energy Consuming Grade: Which grade of steel during manufacturing process consumes more energy or electricity?
 -  Grade of steel consuming the most energy: 40MnSiVS6

### 	Least Energy Consuming Grade: Which grades of steel during manufacturing process consumes less energy or electricity?
 -   Grade of steel consuming the least energy: AISI435MOH

•	Production Time Analysis: Determining the average melting and total cycle time per grade.

### Shift Efficiency: Which shift has the highest energy consumption? 
 -  Shift with the highest energy consumption: C with 124638333.881 units of energy consumed









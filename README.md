# Data-Cleaning-using-Python

![](icon.png)

## Project Overview
This project centers on data cleaning and anomaly detection for a customer dataset, to make it eligible for further analysis. The dataset contains customer information such as personal details, job industry, wealth segment, purchase history, and tenure.

## Objectives
The goal of this project was to clean and prepare a customer dataset by :

Handling missing values

Fixing inconsistent data

Detecting and handling outliers

Performing basic anomaly detection

Ensuring data quality for future use

### Tools

- Python, Jupiter NoteBook

## About Dataset
The dataset contains information on customer demographics and transactions stored in a excel file, having the number of entries and columns stated below;

- Number of Entries: 4000 rows

- Number of Columns: 13

![](dataset.PNG)

Key columns include:

**customer_id** – Unique identifier for each customer

**last_name** – Customer’s last name

**gender** – Categorical field with possible inconsistencies

**job_title** – Customer’s job title

**job_industry_category** – The industry the customer works in

**DOB** – Date of birth

**wealth_segment** - Category of csutoner according to wealth

**tenure** – Number of years as a customer

**past_3_years_bike_related_purchases** – Total purchases in the last 3 years

**owns_car** – Whether the customer owns a car (Yes/No)

**deceased_indicator** – Whether the customer is deceased (Yes/No)

## Data Cleaning Processes
To begin with, the necessary libraries were imported to read the excel file from the directory into a dataframe on jupiter notebook. Then, data qualities were probed by checking datatypes and missing values.

```
# import  libraries
import pandas as pd
import numpy as np

df = pd.read_excel(data.xlsx)

# display the datatypes for each columns
print("Datatypes:\n", df.dtypes)
```
---
![](data-type.PNG)
---
```
# Check for missing values
print("Missing Values:\n", df.isnull().sum())

```
---
![](missing-value-check.PNG)
---
### Handling Missing Data

Columns | Issue | Solution Applied
|-------|-------|-----------------|
|last_name |Missing values |Filled with "Unknown" |
|job_title |Missing values |Filled with "Unknown" |
|job_industry_category |Missing values |Imputed based on most common category per job title |
|DOB |Missing values |Replaced with the median date of birth |
|tenure |Missing values |Replaced with the median tenure |

```
# replace missing values in the text columns ("last_name", "job_title") with "Unknown"
df.fillna({"last_name":"Unknown"}, inplace=True)
df.fillna({"job_title":"Unknown"}, inplace=True)

# Fill job_industry_category based on most common category per job_title
job_industry_mode = df.groupby("job_title")["job_industry_category"].apply(lambda x: x.mode().iloc[0] if not x.mode().empty else "Unknown")
df["job_industry_category"] = df.apply(lambda row: job_industry_mode[row["job_title"]] if pd.isna(row["job_industry_category"]) else row["job_industry_category"], axis=1)

# Ensure correct date format and replace missing values with median DOB
df["DOB"] = pd.to_datetime(df["DOB"], errors="coerce")
median_dob = df["DOB"].median()  # Calculate median DOB
df.fillna({"DOB":median_dob}, inplace = True)

# Replace missing values with median tenure
df.fillna({"tenure" : df["tenure"].median()}, inplace=True)
```

### Fixing Inconsistent Data

- Standardized Gender Values: Inconsistencies in the gender column such as "M", "F", "femal", and "u" were standardized by converting all values to lowercase, trimming whitespaces, and then mapping them to the correct format

- Fixed Spelling Errors:In the job_industry_category column, spelling inconsistencies detected and corrected. For instance, "Argiculture" was corrected to "Agriculture"

- Standardized Boolean Values (owns_car, deceased_indicator): This was done by stripping leading/trailing spaces and capitalizing values to a consistent format.The owns_car and deceased_indicator columns had inconsistent capitalization and formatting (e.g., "YES", "no", "yes ") which were standardized to "Yes", "No" . 

```
# Standardize gender values
df["gender"] = df["gender"].str.strip().str.lower()  # Remove spaces & convert to lowercase
df["gender"] = df["gender"].replace({
    "m": "Male", "male": "Male",
    "f": "Female", "female": "Female",
    "femal": "Female", "u": "Unknown"
})

# Fix spelling error in the column ("job_industry_category")
df["job_industry_category"].unique() # to display values prior to correction
df["job_industry_category"] = df["job_industry_category"].replace({"Argiculture" : "Agriculture"})

# Standardize boolean values in the columns ("owns_car", "deceased_indicator") 
df["owns_car"] = df["owns_car"].str.strip().str.upper()
df["owns_car"] = df["owns_car"].replace({"YES": "Yes", "NO": "No"})

df["deceased_indicator"] = df["deceased_indicator"].str.strip().str.upper()
df["deceased_indicator"] = df["deceased_indicator"].replace({"Y": "Yes", "N": "No"})

# Remove irrelevant column
df.drop("default", axis=1, inplace=True)

df.head()
```
### Detection and Handling of Outliers
It is a pertinent step in data cleaning to look out for outliers in a dataset as this detection allows us to check for values that fall significantly outside the typical range of data. Using **Interquartile Range (IQR) method**, outlier detection was performed on the dataset's numerical columns.

```
# List of numerical columns to check for outliers
numerical_cols = ["past_3_years_bike_related_purchases", "tenure"]

# Function to detect outliers using IQR
def detect_outliers(df, column):
    Q1 = df[column].quantile(0.25)
    Q3 = df[column].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    
    outliers = df[(df[column] < lower_bound) | (df[column] > upper_bound)]
    return outliers

# Check outliers for each numerical column
for col in numerical_cols:
    outliers = detect_outliers(df, col)
    print(f"Outliers in {col}: {len(outliers)}")

```
The above code revealed **no extreme outliers** in the key numerical fields, which indicates that the dataset did not contain unusual or extreme values that could distort results.

### Anomaly Detection
This is aimed at identifying and addressing irregularities that could affect outcomes of further analysis, which involved checking for duplicate records and dropping irrelevant columns from the dataset.

**Duplicate Records:** The unique "customer_id" field was checked for duplicate entries to ensure no customer was recorded multiple times in the dataset. It revealed no duplicate records, confirming the dataset's uniqueness at the entity level

```
# Find duplicate customer records based on "customer_id"
duplicates = df[df.duplicated(subset=["customer_id"], keep=False)]
print(f"Duplicate customer records found: {len(duplicates)}")

```
**Irrelevant Column:** Upon reviewing the dataset for columns that were irrelevant or contained insufficient information. Hence, the "default" column was dropped from the dataset.

```
# Remove irrelevant column
df.drop("default", axis=1, inplace=True)
```

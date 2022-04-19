# streamlined_data_ingestion_with_pandas

***Take a look of my simple work with Data Camp, Ingestion of data from flat files, Excel, Json and APIs***

_________________________________________________________________
### Working with Flat Files

> The version of Vermont tax data here is a tab-separated values file (TSV),  Once the file has been loaded, You have to group the N1 field, which contains income range categories, to create a chart of tax returns by income category.
```
# Import pandas with the alias pd
import pandas as pd

# Load TSV using the sep keyword argument to set delimiter
data = pd.read_csv('vt_tax_data_2016.tsv',sep='\t')
print(data.head())
# Plot the total number of tax returns by income group
counts = data.groupby("agi_stub").N1.sum()
print(counts)
counts.plot.bar()
plt.show()
```
____________________________

> When working with large files, it can be easier to load and process the data in pieces. Let's practice this workflow on the Vermont tax data.
```
# Import pandas with the alias pd
import pandas as pd

# Create dataframe of next 500 rows with labeled columns
vt_data_next500 = pd.read_csv("vt_tax_data_2016.csv", nrows=500, skiprows=500,header=None,names=list(vt_data_first500))

# View the Vermont dataframes to confirm they're different
print(vt_data_first500.head())
print(vt_data_next500.head())
```
____________________________

> Looking at the data dictionary for vt_tax_data_2016.csv reveals two such columns. The agi_stub column contains numbers that correspond to income categories, and zipcode has 5-digit values that should be strings -- treating them as integers means we lose leading 0s, which are meaningful. Let's specify the correct data type.
```
# Import pandas with the alias pd
import pandas as pd

# Create dict specifying data types for agi_stub and zipcode
data_types = {"agi_stub": "category",
			  "zipcode": str}

# Load csv using dtype to set correct data types
data = pd.read_csv("vt_tax_data_2016.csv", dtype=data_types)

# Print data types of resulting frame
print(data.dtypes.head())
```
____________________________

> we can pass additional NA indicators with the na_values argument.
```
# Import pandas with the alias pd
import pandas as pd

#Create a dictionary, null_values, specifying that 0s in the zipcode column should be considered NA values.
null_values = {"zipcode" : 0}

# Load csv using na_values keyword argument
data = pd.read_csv("vt_tax_data_2016.csv", 
                   na_values=null_values)

# View rows with NA ZIP codes
print(data[data.zipcode.isna()])
#print(data.zipcode.isna())
```
____________________________

> Some lines in the Vermont tax data here are corrupted. In order to load the good lines, we need to tell pandas to skip errors.
```
# Import pandas with the alias pd
import pandas as pd

try:
  # Set warn_bad_lines to issue warnings about bad records
  data = pd.read_csv("vt_tax_data_2016_corrupt.csv", 
                     error_bad_lines=False, 
                     warn_bad_lines=True)
  
  # View first 5 records
  print(data.head())
  
except pd.io.common.CParserError:
    print("Your data contained rows that could not be parsed.")
```
____________________________

### Importing Data from Excel Files
```
#Load pandas as pd
import pandas as pd

#Read spreadsheet and assign it to survey_responses
survey_responses = pd.read_excel('fcc_survey.xlsx')

#View the head of the dataframe
print(survey_responses.head())
```
____________________________

### Load a portion of a spreadsheet

```
#Load pandas as pd
import pandas as pd

# Create string of lettered columns to load
col_string = 'AD, AW:BA'

# Load data with skiprows and usecols set
survey_responses = pd.read_excel("fcc_survey_headers.xlsx", 
                        skiprows=2, 
                        usecols=col_string)

# View the names of the columns selected
print(survey_responses.columns)
```

____________________________

> An Excel workbook may contain multiple sheets of related data. The New Developer Survey response workbook has sheets for different years. Because read_excel() loads only the first sheet by default, you've already gotten survey responses for 2016. Now, Create a dataframe of 2017 responses.

### Select a single sheet
```
import pandas as pd
# Create df from second worksheet by referencing its position
responses_2017 = pd.read_excel("fcc_survey.xlsx",sheet_name=1)
                               
# by sheet name
# responses_2017 = pd.read_excel("fcc_survey.xlsx",sheet_name='2017')

# Graph where people would like to get a developer job
job_prefs = responses_2017.groupby("JobPref").JobPref.count()
job_prefs.plot.barh()
plt.show()
```
____________________________
### Work with Multiples sheets

> Load both the 2016 and 2017 sheets

```
import pandas as pd
all_survey_data = pd.read_excel("fcc_survey.xlsx",sheet_name=None)
# all_survey_data = pd.read_excel("fcc_survey.xlsx",sheet_name=['2016','2017'])
# all_survey_data = pd.read_excel("fcc_survey.xlsx", sheet_name=[0,'2017'])

#  View the sheet names in all_survey_data
print(all_survey_data.keys())
# View the data type of all_survey_data
print(type(all_survey_data))
```

> The FreeCodeCamp New Developer Survey file is set up similarly, with samples of ***responses*** from different years in different sheets, All sheets have been read into the ordered dictionary responses, where sheet names are keys and dataframes are values, Your task here is to compile them in one dataframe for analysis.
```
import pandas as pd
# Create an empty dataframe
all_responses = pd.DataFrame()

# Set up for loop to iterate through values in responses
for df in responses.values():
  # Print the number of rows being added
  print("Adding {} rows".format(df.shape[0]))
  
  # Append df to all_responses, assign result
  all_responses = all_responses.append(df)

# Graph employment statuses in sample
counts = all_responses.groupby("EmploymentStatus").EmploymentStatus.count()
counts.plot.barh()
plt.show()
```
____________________________

> since defaulting to Booleans may have undesired effects like turning NA values into Trues; fcc_survey_subset.xlsx contains a string ID column and several True/False columns indicating financial stressors. You'll evaluate which non-ID columns have no NA values and therefore can be set as Boolean, then tell read_excel() to load them as such with the dtype argument.
```
# Import pandas with the alias pd
import pandas as pd

# Set dtype to load appropriate column(s) as Boolean data
survey_data = pd.read_excel("fcc_survey_subset.xlsx",
                            dtype={"HasDebt":bool})
#print(survey_data.head(25))
# View financial burdens by Boolean group
print(survey_data.groupby('HasDebt').sum())
```
____________________________

> Some datasets, like survey data, can use unrecognized values, such as "Yes" and "No", For practice purposes, some Boolean columns in the New Developer Survey have been coded this way. You'll make sure they're properly interpreted
```
# Import pandas with the alias pd
import pandas as pd

# Load file with Yes as a True value and No as a False value
survey_subset = pd.read_excel("fcc_survey_yn_data.xlsx",
                              dtype={"HasDebt": bool,
                              "AttendedBootCampYesNo": bool},
                              true_values=['yes','Yes','YES'],
                              false_values=['NO','no','No'])

# View the data
print(survey_subset.head())
```
____________________________

> Sometimes, datetime data is split across columns. A dataset might have a date and a time column, or a date may be split into year, month, and day columns. Your task is combining them into one datetime column with a new name.
```
# Import pandas with the alias pd
import pandas as pd

# Create dict of columns to combine into new datetime column
datetime_cols = {"Part2Start": ['Part2StartDate','Part2StartTime']}


# Load file, supplying the dict to parse_dates
survey_data = pd.read_excel("fcc_survey_dts.xlsx",
                            parse_dates=datetime_cols)

# View summary statistics about Part2Start
print(survey_data.Part2Start.describe())
```
____________________________

> if a date is in a non-standard format, like 19991231 for December 31, 1999, it can't be parsed at the import stage, to convert it, passing in the column to convert and a string representing the date format used.

```
# Import pandas with the alias pd
import pandas as pd

# Load file, supplying the dict to parse_dates
survey_data = pd.read_excel("fcc_survey_dts.xlsx")

# Parse datetimes and assign result back to Part2EndTime
survey_data["Part2EndTime"] = pd.to_datetime(survey_data["Part2EndTime"], 
                                             format="%m%d%Y %H:%M:%S")

# Print first few values of Part2EndTime
print(survey_data.Part2EndTime.head())
```
____________________________

### Working With Jsons 

> Get a sense of the contents of dhs_daily_report.json
```
# Import pandas with the alias pd
import pandas as pd

# Load the daily report to a dataframe
pop_in_shelters = pd.read_json('dhs_daily_report.json')

# View summary stats about pop_in_shelters
print(pop_in_shelters.describe())
```

____________________________

> JSON isn't a tabular format, so pandas makes assumptions about its orientation when loading data. Most JSON data you encounter will be in orientations that pandas can automatically transform into a dataframe.
```
# Import pandas with the alias pd
import pandas as pd

try:
    # Load the JSON with orient specified
    df = pd.read_json("dhs_report_reformatted.json",
                      orient='split')
    
    # Plot total population in shelters over time
    df["date_of_census"] = pd.to_datetime(df["date_of_census"])
    df.plot(x="date_of_census", 
            y="total_individuals_in_shelter")
    plt.show()
    
except ValueError:
    print("pandas could not parse the JSON.")
```
____________________________

> The Yelp API requires the location parameter be set. It also lets users supply a term to search for. You'll use these parameters to get data about cafes in NYC, then process the result to create a dataframe.
```
# Import pandas with the alias pd
import pandas as pd
import requests

api_url = "https://api.yelp.com/v3/businesses/search"

# Create dictionary to query API for cafes in NYC
parameters = {"term": "cafe",
          	  "location": "NYC"}

# Query the Yelp API with headers and params set
response = requests.get(api_url, 
                        headers=headers, 
                        params=parameters)

# Extract JSON data from response
data = response.json()

# Load "businesses" values to a data frame and print head
cafes = pd.DataFrame(data["businesses"])
print(cafes.head())
```
____________________________

> The categories attribute in the Yelp API response contains lists of objects. To flatten this data, you'll employ json_normalize() arguments to specify the path to categories and pick other attributes to include in the dataframe. You should also change the separator to facilitate column selection and prefix the other attributes to prevent column name collisions.

```
# Load json_normalize()
# Import pandas with the alias pd
import pandas as pd
import requests
from pandas.io.json import json_normalize

api_url = "https://api.yelp.com/v3/businesses/search"

# Create dictionary to query API for cafes in NYC
parameters = {"term": "cafe",
          	  "location": "NYC"}

# Query the Yelp API with headers and params set
response = requests.get(api_url, 
                        headers=headers, 
                        params=parameters)

# Extract JSON data from response
data = response.json()

# Load other business attributes and set meta prefix
flat_cafes = json_normalize(data["businesses"],sep="_",record_path="categories",
meta=['name', 'alias', 'rating', ['coordinates', 'latitude'], ['coordinates', 'longitude']],meta_prefix='biz_') 
                                                           
# View the data
print(flat_cafes.head())
```
________________________________________________

- Sort homelessness by the number of homeless individuals, from smallest to largest, and save this as homelessness_ind.
- Print the head of the sorted DataFrame.
- Sort homelessness by the number of homeless family_members in descending order, and save this as homelessness_fam.
- Print the head of the sorted DataFrame. 
- Sort homelessness first by region (ascending), and then by number of family members (descending). Save this as homelessness_reg_fam.
- Print the head of the sorted DataFrame.
- Create a DataFrame called individuals that contains only the individuals column of homelessness.
- Print the head of the result.
- Create a DataFrame called state_fam that contains only the state and family_members columns of homelessness, in that order.
- Print the head of the result. 
- Create a DataFrame called ind_state that contains the individuals and state columns of homelessness, in that order.
- Print the head of the result.
- Filter homelessness for cases where the number of individuals is greater than ten thousand, assigning to ind_gt_10k. View the printed result.
- Filter homelessness for cases where the USA Census region is "Mountain", assigning to mountain_reg. View the printed result.
- Filter homelessness for cases where the number of family_members is less than one thousand and the region is "Pacific", assigning to fam_lt_1k_pac. View the printed result.
- Filter homelessness for cases where the USA census region is "South Atlantic" or it is "Mid-Atlantic", assigning to south_mid_atlantic. View the printed result.
- Filter homelessness for cases where the USA census state is in the list of Mojave states, canu, assigning to mojave_homelessness. View the printed result.
- Add a new column to homelessness, named total, containing the sum of the individuals and family_members columns.
- Add another column to homelessness, named p_individuals, containing the proportion of homeless people in each state who are individuals.
- Add a column to homelessness, indiv_per_10k, containing the number of homeless individuals per ten thousand people in each state.
- Subset rows where indiv_per_10k is higher than 20, assigning to high_homelessness.
- Sort high_homelessness by descending indiv_per_10k, assigning to high_homelessness_srt.
- Select only the state and indiv_per_10k columns of high_homelessness_srt and save as result. Look at the result.

```
# Load pandas as pd
import pandas as pd


# Sort homelessness by individuals
homelessness_ind = homelessness.sort_values('individuals')

# Print the top few rows
print(homelessness_ind.head())

# Sort homelessness by descending family members
homelessness_fam = homelessness.sort_values('family_members', ascending=False)

# Print the top few rows
print(homelessness_fam.head())

# Sort homelessness by region, then descending family members
homelessness_reg_fam = homelessness.sort_values(['region',"family_members"],ascending=[True,False])

# Print the top few rows
print(homelessness_reg_fam.head())

# Select the individuals column
individuals = homelessness['individuals']

# Print the head of the result
print(individuals.head())

# Select the state and family_members columns
state_fam = homelessness[['state',"family_members"]]

# Print the head of the result
print(state_fam.head())

# Select only the individuals and state columns, in that order
ind_state = homelessness[['individuals',"state"]]

# Print the head of the result
print(ind_state.head())


# Filter for rows where individuals is greater than 10000
ind_gt_10k = homelessness[homelessness['individuals']>10000]

# See the result
print(ind_gt_10k)

# Filter for rows where region is Mountain
mountain_reg = homelessness[homelessness['region']=='Mountain']

# See the result
print(mountain_reg)

# Filter for rows where family_members is less than 1000 
# and region is Pacific
fam_lt_1k_pac = homelessness[(homelessness['family_members']<1000) & (homelessness['region']=='Pacific')]

# See the result
print(fam_lt_1k_pac)

# Subset for rows in South Atlantic or Mid-Atlantic regions
south_mid_atlantic = homelessness[(homelessness["region"] == "South Atlantic") | (homelessness["region"] == "Mid-Atlantic")]

# See the result
print(south_mid_atlantic)

# The Mojave Desert states
canu = ["California", "Arizona", "Nevada", "Utah"]

# Filter for rows in the Mojave Desert states
mojave_homelessness = homelessness[homelessness["state"].isin(canu)]

# See the result
print(mojave_homelessness)

# Add total col as sum of individuals and family_members
homelessness["total"] = homelessness["individuals"] + homelessness["family_members"]

# Add p_individuals col as proportion of total that are individuals
homelessness["p_individuals"] = homelessness["individuals"] / homelessness["total"]

# See the result
print(homelessness)

# Create indiv_per_10k col as homeless individuals per 10k state pop
homelessness["indiv_per_10k"] = 10000 * homelessness["individuals"] / homelessness["state_pop"] 

# Subset rows for indiv_per_10k greater than 20
high_homelessness = homelessness[homelessness["indiv_per_10k"] > 20]

# Sort high_homelessness by descending indiv_per_10k
high_homelessness_srt = high_homelessness.sort_values("indiv_per_10k", ascending=False)

# From high_homelessness_srt, select the state and indiv_per_10k cols
result = high_homelessness_srt[["state", "indiv_per_10k"]]

# See the result
print(result)

```
_______________________________________________________________________________

> Create function Inter-quartile range, which is the 75th percentile minus the 25th percentile.

- Use your function defined for you along with .agg() to print the Inter-quartile range of the temperature_c, fuel_price_usd_per_l and columns of sales.
- Update the aggregation functions called by .agg(): include Inter-quartile range and np.median in that order.
```
# Load pandas as pd
import pandas as pd

# A custom IQR function
def iqr(column):
    return column.quantile(0.75) - column.quantile(0.25)

# Update to print IQR of temperature_c, fuel_price_usd_per_l, & unemployment
print(sales[["temperature_c", 'fuel_price_usd_per_l', 'unemployment']].agg(iqr))
```
_______________________________________________________________________________
### Cumulative statistics

- Sort the rows of sales_1_1 by the date column in ascending order.
- Get the cumulative sum of weekly_sales and add it as a new column of sales_1_1 called cum_weekly_sales.
- Get the cumulative maximum of weekly_sales, and add it as a column called cum_max_sales.
- Print the date, weekly_sales, cum_weekly_sales, and cum_max_sales columns.
```
# Load pandas as pd
import pandas as pd

# Sort sales_1_1 by date
sales_1_1 = sales_1_1.sort_values('date')

# Get the cumulative sum of weekly_sales, add as cum_weekly_sales col
sales_1_1['cum_weekly_sales'] = sales_1_1['weekly_sales'].cumsum()

# Get the cumulative max of weekly_sales, add as cum_max_sales col
sales_1_1['cum_max_sales'] = sales_1_1['weekly_sales'].cummax()


# See the columns you calculated
print(sales_1_1[["date", "weekly_sales", "cum_weekly_sales", "cum_max_sales"]])
```
_______________________________________________________________________________

- Remove rows of sales with duplicate pairs of store and type and save as store_types and print the head.
- Remove rows of sales with duplicate pairs of store and department and save as store_depts and print the head.
- Subset the rows that are holiday weeks using the is_holiday column, and drop the duplicate dates, saving as holiday_dates.
- Select the date column of holiday_dates, and print.
- Count the number of stores of each store type in store_types.
- Count the proportion of stores of each store type in store_types.
- Count the number of different departments in store_depts, sorting the counts in descending order.
- Count the proportion of different departments in store_depts, sorting the proportions in descending order.
```
# Load pandas as pd
import pandas as pd

# Drop duplicate store/type combinations
store_types = sales.drop_duplicates(subset=['store','type'])
print(store_types.head())

# Drop duplicate store/department combinations
store_depts = sales.drop_duplicates(subset=['store','department'])
print(store_depts.head())

# Subset the rows where is_holiday is True and drop duplicate dates
holiday_dates = sales[sales['is_holiday']].drop_duplicates(subset='date')

# Print date col of holiday_dates
print(holiday_dates['date'].head())

# Count the number of stores of each type
store_counts = store_types["type"].value_counts()
print(store_counts)

# Get the proportion of stores of each type
store_props = store_types["type"].value_counts(normalize=True)
print(store_props)

# Count the number of each department number and sort
dept_counts_sorted = store_depts["department"].value_counts(sort=True)
print(dept_counts_sorted)

# Get the proportion of departments of each number and sort
dept_props_sorted = store_depts["department"].value_counts(sort=True, normalize=True)
print(dept_props_sorted)

```
_______________________________________________________________________________

- Calculate the total weekly_sales over the whole dataset.
- Subset for type "A" stores, and calculate their total weekly sales.
- Do the same for type "B" and type "C" stores.
- Combine the A/B/C results into a list, and divide by sales_all to get the proportion of sales by type.
- Group sales by "type", take the sum of "weekly_sales", and store as sales_by_type.
- Calculate the proportion of sales at each store type by dividing by the sum of sales_by_type. Assign to sales_propn_by_type.
- Group sales by "type" and "is_holiday", take the sum of weekly_sales, and store as sales_by_type_is_holiday.
- Get the min, max, mean, and median of weekly_sales for each store type using .groupby() and .agg(). Store this as sales_stats. Make sure to use numpy functions!
- Get the min, max, mean, and median of unemployment and fuel_price_usd_per_l for each store type. Store this as unemp_fuel_stats.
```
# Imports
import pandas as pd
import numpy as np

#print(sales.head())
# Calc total weekly sales
sales_all = sales["weekly_sales"].sum()
#print(sales_all)
# Subset for type A stores, calc total weekly sales
sales_A = sales[sales["type"] == "A"]["weekly_sales"].sum()
#print(sales_A)
# Subset for type B stores, calc total weekly sales
sales_B = sales[sales["type"] == "B"]["weekly_sales"].sum()
#print(sales_B)

# Subset for type C stores, calc total weekly sales
sales_C = sales[sales["type"] == "C"]["weekly_sales"].sum()
#print(sales_C)

# Get proportion for each type
sales_propn_by_type = [sales_A, sales_B, sales_C] / sales_all
print(sales_propn_by_type)

# Group by type; calc total weekly sales
sales_by_type = sales.groupby("type")["weekly_sales"].sum()

# Get proportion for each type
sales_propn_by_type = sales_by_type / sum(sales_by_type)
print(sales_propn_by_type)

# From previous step
sales_by_type = sales.groupby("type")["weekly_sales"].sum()

# Group by type and is_holiday; calc total weekly sales
sales_by_type_is_holiday = sales.groupby(["type",'is_holiday'])["weekly_sales"].sum()
print(sales_by_type_is_holiday)

# For each store type, aggregate weekly_sales: get min, max, mean, and median
sales_stats = sales.groupby("type")["weekly_sales"].agg([np.min, np.max, np.mean, np.median])

# Print sales_stats
print(sales_stats)

# For each store type, aggregate unemployment and fuel_price_usd_per_l: get min, max, mean, and median
unemp_fuel_stats = sales.groupby("type")[["unemployment", "fuel_price_usd_per_l"]].agg([np.min, np.max, np.mean, np.median])

# Print unemp_fuel_stats
print(unemp_fuel_stats)
```
_______________________________________________________________________________

- Get the mean weekly_sales by type using .pivot_table() and store as mean_sales_by_type.
- Get the mean and median (using NumPy functions) of weekly_sales by type using .pivot_table() and store as mean_med_sales_by_type.
- Get the mean of weekly_sales by type and is_holiday using .pivot_table() and store as mean_sales_by_type_holiday.
- Print the mean weekly_sales by department and type, filling in any missing values with 0.
- Print the mean weekly_sales by department and type, filling in any missing values with 0 and summing all rows and columns.
```
# Imports
import pandas as pd
import numpy as np

# Pivot for mean weekly_sales for each store type
mean_sales_by_type = sales.pivot_table(values='weekly_sales',index='type')

# Print mean_sales_by_type
print(mean_sales_by_type)

# Import NumPy as np
import numpy as np

# Pivot for mean and median weekly_sales for each store type
mean_med_sales_by_type = sales.pivot_table(values="weekly_sales", index="type", aggfunc=[np.mean, np.median])

# Print mean_med_sales_by_type
print(mean_med_sales_by_type)

# Pivot for mean weekly_sales by store type and holiday 
mean_sales_by_type_holiday = sales.pivot_table(values="weekly_sales", index="type", columns="is_holiday")

# Print mean_sales_by_type_holiday
print(mean_sales_by_type_holiday)

# Print mean weekly_sales by department and type; fill missing values with 0
print(sales.pivot_table(values='weekly_sales',index='department',columns='type',fill_value=0))

# Print the mean weekly_sales by department and type; fill missing values with 0s; sum all rows and cols
print(sales.pivot_table(values="weekly_sales", index="department", columns="type", fill_value=0,margins=True))
```
_______________________________________________________________________________

- Look at temperatures.
- Set the index of temperatures to "city", assigning to temperatures_ind.
- Look at temperatures_ind. How is it different from temperatures?
- Reset the index of temperatures_ind, keeping its contents.
- Reset the index of temperatures_ind, dropping its contents.
- Create a list called cities that contains "Moscow" and "Saint Petersburg".
- Use [] subsetting to filter temperatures for rows where the city column takes a value in the cities list.
- Use .loc[] subsetting to filter temperatures_ind for rows where the city is in the cities list.
- Set the index of temperatures to the "country" and "city" columns, and assign this to temperatures_ind.
- Specify two country/city pairs to keep: "Brazil"/"Rio De Janeiro" and "Pakistan"/"Lahore", assigning to rows_to_keep.
- Print and subset temperatures_ind for rows_to_keep using .loc[].
- Sort temperatures_ind by the index values.
- Sort temperatures_ind by the index values at the "city" level.
- Sort temperatures_ind by ascending country then descending city.

```
# Imports
import pandas as pd

# Look at temperatures
print(temperatures.head())

# Index temperatures by city
temperatures_ind = temperatures.set_index('city')

# Look at temperatures_ind
print(temperatures_ind.head())

# Reset the index, keeping its contents
print(temperatures_ind.reset_index())

# Reset the index, dropping its contents
print(temperatures_ind.reset_index(drop=True))
# Make a list of cities to subset on
cities = ["Moscow", "Saint Petersburg"]

# Subset temperatures using square brackets
print(temperatures[temperatures["city"].isin(cities)])

# Subset temperatures_ind using .loc[]
print(temperatures_ind.loc[cities])

# Index temperatures by country & city
temperatures_ind = temperatures.set_index(['country','city'])

# List of tuples: Brazil, Rio De Janeiro & Pakistan, Lahore
rows_to_keep = [('Brazil','Rio De Janeiro'),('Pakistan','Lahore')]

# Subset for rows to keep
print(temperatures_ind.loc[(rows_to_keep)])

# Sort temperatures_ind by index values
print(temperatures_ind.sort_index())

# Sort temperatures_ind by index values at the city level
print(temperatures_ind.sort_index(level="city"))

# Sort temperatures_ind by country then descending city
print(temperatures_ind.sort_index(level=["country", "city"], ascending = [True, False]))

```
_______________________________________________________________________________

- Sort the index of temperatures_ind.
- Use slicing with .loc[] to get these subsets:
	- from Pakistan to Russia.
	- from Lahore to Moscow. (This will return nonsense.)
	- from Pakistan, Lahore to Russia, Moscow.
- Use .loc[] slicing to subset rows from India, Hyderabad to Iraq, Baghdad.
- Use .loc[] slicing to subset columns from date to avg_temp_c.
- Slice in both directions at once from Hyderabad to Baghdad, and date to avg_temp_c.
- Use Boolean conditions, not .isin() or .loc[], and the full date "yyyy-mm-dd", to subset temperatures for rows in 2010 and 2011 and print the results.
- Set the index of temperatures to the date column and sort it.
- Use .loc[] to subset temperatures_ind for rows in 2010 and 2011.
- Use .loc[] to subset temperatures_ind for rows from Aug 2010 to Feb 2011.
- Get the 23rd row, 2nd column (index positions 22 and 1).
- Get the first 5 rows (index positions 0 to 5).
- Get all rows, columns 3 and 4 (index positions 2 to 4).
- Get the first 5 rows, columns 3 and 4.


```
# Imports
import pandas as pd

# Sort the index of temperatures_ind
temperatures_srt = temperatures_ind.sort_index()

# Subset rows from Pakistan to Russia
print(temperatures_srt.loc["Pakistan":"Russia"])

# Try to subset rows from Lahore to Moscow
print(temperatures_srt.loc["Lahore":"Moscow"])

# Subset rows from Pakistan, Lahore to Russia, Moscow
print(temperatures_srt.loc[("Pakistan", "Lahore"):("Russia", "Moscow")])

# Subset rows from India, Hyderabad to Iraq, Baghdad
print(temperatures_srt.loc[("India", "Hyderabad"):("Iraq", "Baghdad")])

# Subset columns from date to avg_temp_c
print(temperatures_srt.loc[:, "date":"avg_temp_c"])

# Subset in both directions at once
print(temperatures_srt.loc[("India", "Hyderabad"):("Iraq", "Baghdad"), "date":"avg_temp_c"])

# Use Boolean conditions to subset temperatures for rows in 2010 and 2011
temperatures_bool = temperatures[(temperatures["date"] >= "2010-01-01") & (temperatures["date"] <= "2011-12-31")]
print(temperatures_bool)

# Set date as the index and sort the index
temperatures_ind = temperatures.set_index("date").sort_index()

# Use .loc[] to subset temperatures_ind for rows in 2010 and 2011
print(temperatures_ind.loc["2010":"2011"])

# Use .loc[] to subset temperatures_ind for rows from Aug 2010 to Feb 2011
print(temperatures_ind.loc["2010-08":"2011-02"])

# Get 23rd row, 2nd column (index 22, 1)
print(temperatures.iloc[22, 1])

# Use slicing to get the first 5 rows
print(temperatures.iloc[:5])

# Use slicing to get columns 3 to 4
print(temperatures.iloc[:, 2:4])

# Use slicing in both directions at once
print(temperatures.iloc[:5, 2:4])
```
_______________________________________________________________________________

### Visualizing
- Print the head of the avocados dataset. What columns are available?
- For each avocado size group, calculate the total number sold, storing as nb_sold_by_size.
- Create a bar plot of the number of avocados sold by size.
- Show the plot.
- Get the total number of avocados sold on each date. The DataFrame has two rows for each date—one for organic, and one for conventional. Save this as nb_sold_by_date.
- Create a line plot of the number of avocados sold.
- Show the plot.
- Create a scatter plot with nb_sold on the x-axis and avg_price on the y-axis. Title it "Number of avocados sold vs. average price".
- Show the plot.
- Subset avocados for the conventional type, and the average price column. Create a histogram.
- Create a histogram of avg_price for organic type avocados.
- Add a legend to your plot, with the names "conventional" and "organic".
- Show your plot.
- Modify your code to adjust the transparency of both histograms to 0.5 to see how much overlap there is between the two distributions.
- Modify your code to use 20 bins in both histograms.

```
# Imports
import pandas as pd
import matplotlib.pyplot as plt

# Scatter plot of avg_price vs. nb_sold with title
avocados.plot(x="nb_sold", y="avg_price", kind="scatter", title="Number of avocados sold vs. average price")

# Show the plot
plt.show()

# Modify histogram transparency to 0.5
avocados[avocados["type"] == "conventional"]["avg_price"].hist(alpha=0.5)

# Modify histogram transparency to 0.5
avocados[avocados["type"] == "organic"]["avg_price"].hist(alpha=0.5)

# Add a legend
plt.legend(["conventional", "organic"])

# Show the plot
plt.show()

# Modify bins to 20
avocados[avocados["type"] == "conventional"]["avg_price"].hist(alpha=0.5, bins=20)

# Modify bins to 20
avocados[avocados["type"] == "organic"]["avg_price"].hist(alpha=0.5, bins=20)

# Add a legend
plt.legend(["conventional", "organic"])

# Show the plot
plt.show()
```
_______________________________________________________________________________


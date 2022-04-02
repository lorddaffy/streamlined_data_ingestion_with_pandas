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
flat_cafes = json_normalize(data["businesses"],sep="_",record_path="categories",meta=['name', 'alias', 'rating', ['coordinates', 'latitude'], ['coordinates', 'longitude']],meta_prefix='biz_') 
                                                           
# View the data
print(flat_cafes.head())
```

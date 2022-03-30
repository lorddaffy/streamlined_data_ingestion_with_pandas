# streamlined_data_ingestion_with_pandas
<span style="color:blue">some *Take a look of my simple work with Data Camp* text</span>


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

# streamlined_data_ingestion_with_pandas
Take a look of my simple work with Data Camp

### Importing Data from Excel Files
```
#Load pandas as pd
import pandas as pd

#Read spreadsheet and assign it to survey_responses
survey_responses = pd.read_excel('fcc_survey.xlsx')

#View the head of the dataframe
print(survey_responses.head())
```

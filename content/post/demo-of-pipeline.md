---
title: Demo of Azure Normalize Pipeline"
date: 2021-03-09T21:19:02-05:00
draft: false
categories:
  -  ["Projects"]
tags:
  - ["Duke",
    "CovIdentify",
    "themes",
    "development"]
---

# Demo of AzureNormalizePipeline
Last edited: Mar 05, 2021

## Description
This function was build in support of the BIGIDEAS Lab's Covidentify Project (Duke University). The intention of the function is to extract, transform, and load the unstructured json data from the database that houses the Fitbit and Garmin API pulls into Microsoft Azure's ML Studio. 

## Background
   Because of the nature of Duke's Internal Review Board (IRB), the lab does not have direct control over the API pulls and thus using noSQL was not an option at this time. Without transformation, the json data was unusable to perform any modeling. The entire project was migrated to Azure during the month of Dec 2021, and this package was written to help transform and load the data for use in a Covid detection model.

## How to import the package


```python
from AzureNormalizePipeline import *
```

## Load the data from a Azure Dataset


```python
# azureml-core of version 1.0.72 or higher is required
# azureml-dataprep[pandas] of version 1.1.34 or higher is required
from azureml.core import Workspace, Dataset

subscription_id = '9ec123f2-bbdb-4786-bdf0-22f18be7fa34'
resource_group = 'ml_covidentify'
workspace_name = 'covidentify_analysis'

workspace = Workspace(subscription_id, resource_group, workspace_name)

dataset = Dataset.get_by_name(workspace, name='covid_participant_positive')

### OPTIONAL: the function can be used with an Azure Dataset or with a pandas DataFrame (pandas is recommended) ###
df = dataset.to_pandas_dataframe()
```

Within the <code>AzureNormalizePipeline</code> package there are two functions: 
* the main <code>json_normalizer</code> function: performs the transformation and loading work
* <code>whichPromptNames</code> function: provides a list of possible prompt names for the user to look at and iterate over
 
A demo of the functionality is shown below.

## Demo of <code>whichPromptNames</code>

From <code>whichPromptNames</code>'s documentation:
>This function is to help users identify which prompt names are available in json_normalizer function. This is manually inputted and if new prompt names are added, this needs to be updated.
>
>**INPUTS**<br>
><code>device</code>: either "garmin" or "fitbit".<br>
><code>timePeriod</code>: Either "historical" or "current". When users enrolled into study, their "historical" data was shared, "current" data is collected and uploaded to the prompt database.
>
>**OUTPUTS**<br>
>Outputs a list of prompt names.


```python
# prompt = whichPromptNames("garmin", "historical") <- not incorporated yet
# print(">>> garmin, historical:", prompt)
prompt = whichPromptNames("garmin", "current")
print("\n>>> garmin, current:", prompt)
prompt = whichPromptNames("fitbit", "historical")
print("\n>>> fitbit, historical:", prompt)
prompt = whichPromptNames("fitbit", "current")
print("\n>>> fitbit, current:", prompt)
```

    
    >>> garmin, current: ['covid_garmin_activity', 'covid_garmin_daily', 'covid_garmin_epoch', 'covid_garmin_sleep']
    
    >>> fitbit, historical: ['covid_fitbit_heart_rate_intraday_backfill', 'covid_fitbit_steps_intraday_backfill', 'covid_fitbit_floors_intraday_backfill', 'covid_fitbit_floors_intraday_backfill', 'covid_fitbit_elevation_intraday_backfill', 'covid_fitbit_distance_intraday_backfill', 'covid_fitbit_calories_intraday_backfill', 'covid_fitbit_sleep_intraday_backfill']
    
    >>> fitbit, current: ['heart_rate_intraday', 'sleep', 'calories_intraday', 'distance_intraday', 'elevation_intraday', 'floors_intraday', 'steps_intraday', 'r15_daily_sleep', 'weekly_r15_sleep_api', 'weekly_r15_water_api', 'weekly_r15_food_api']


> Using these lists, a user can use a for-loop to iterate over the possible prompt names to quickly generate the needed output datasets.

## Demo of <code>json_normalizer</code>

This code is built off of **Pandas'** <code>json_normalize</code>, but because of the nature of our nested jsons and exisitng relational data, more transformation work had to be incorporated before normalization could occur. Additonally, because each promptName has a different json structure, different keyword arguments needed to be implemented for each promptName to make a process that was generalizable for the entire data.

From <code>json_normalizer</code>'s documentation:
>This function will extract an Azure Dataset from a Azure Dataset Consume\
>transform the nested json into a relational table and load the transformed\
>table directly into the Azure dataSets Assets list.
>
>**INPUTS**<br>
><code>device</code>: either "garmin" or "fitbit"<br>
><code>timePeriod</code>: Either "historical" or "current". When users enrolled into study, their "historical" data was shared, "current" data is collected and uploaded to the prompt database.<br>
><code>promptName</code>: name of the type of data collected.<br>
><code>data</code>: dataset from the targeted Azure DataSet Consume.<br>
><code>aggSummary</code> (optional): for some garmin promptNames, it makes more sense to display the data in the aggregated summaries form, which is one level above the default summaries.<br>
><code>ignoreInputs</code> (optional): if turned on, skips the default user inputs.<br>
><code>prefix</code> (optional): used in conjunction when 'ignoreInputs' is turned on to set the uploaded filename's prefix.<br>
><code>pandas</code> (optional): Allows using a pandas dataFrame instead of using a Azure dataset. There is a performance boost when converting to a pandas dataFrame outside of the function.<br>
>
>**OUTPUTS**<br>
>Automatically uploads the transformed data into the Azure dataSets Assets list.

### json_normalizer, Demo 1: Running the function from a Pandas dataframe


```python
json_normalizer("fitbit", "current", prompt[1], data=df, pandas=True)
```

    >>> The shape of the output dataframe is:  (135, 7)
    >>> The number of rows that did not have data are: 0


    Do you want see a sample? [y/n] y


       sleep captured_at  summary.totalMinutesAsleep  summary.totalSleepRecords  \
    30    []  2020-08-13                           0                          0   
    8     []  2020-07-23                           0                          0   
    14    []  2020-07-29                           0                          0   
    14    []  2020-07-29                           0                          0   
    58    []  2020-09-10                           0                          0   
    ..   ...         ...                         ...                        ...   
    77    []  2020-10-04                           0                          0   
    60    []  2020-09-12                           0                          0   
    58    []  2020-09-10                           0                          0   
    69    []  2020-10-03                           0                          0   
    31    []  2020-08-17                           0                          0   
    
        summary.totalTimeInBed  uniqueRowID  participant_id  
    30                       0      1630190            9032  
    8                        0      1366764            9032  
    14                       0      1427570            9032  
    14                       0      1427570            9032  
    58                       0      2020673            9032  
    ..                     ...          ...             ...  
    77                       0      2362473            9032  
    60                       0      2046203            9032  
    58                       0      2020673            9032  
    69                       0      2353843            9032  
    31                       0      1677049            9032  
    
    [1350 rows x 7 columns]


    Do you want to upload? [y/n] y
    What do you want the prefix for the file to be? demo


    fitbit current sleep aggSummary = False
    ./output_csv/demo-sleep.csv
    Uploading an estimated of 1 files
    Uploading ./output_csv/demo-sleep.csv
    Uploaded ./output_csv/demo-sleep.csv, 1 files out of an estimated total of 1
    Uploaded 1 files
    >>> Upload Successful!


>**Notice that there are user input options above. These user inputs are:**<br>
>   1. Seeing a random sample of 10 lines of the data before uploading. If there are less than 10 lines, the data will be randomly repeated.
>   2. An option to upload the data.
>   3. If uploading, a prefix can be designated to help distinguish the source of the data.

> **Below is a screenshot of Azure's ML Studio to show where the dataset ends up:**<br>

<center><img src="./50_assets/dataset_in_MLPortal.png" alt="Dataset location in portal" style="width: 1000px;"/></center>

> **This screenshot shows that the code auto-generates the description with the input info:**<br>

<center><img src="./50_assets/dataset_description.png" alt="Dataset description" style="width: 1000px;"/></center>

### json_normalizer, Demo 2: What happens if you try to use the wrong inputs


```python
json_normalizer("fitbit", "historical", prompt[1], data=df, pandas=True)
```


    ---------------------------------------------------------------------------

    Exception                                 Traceback (most recent call last)

    <ipython-input-5-e3878ef1b7d9> in <module>
    ----> 1 json_normalizer("fitbit", "historical", prompt[1], data=df, pandas=True)
    

    /mnt/batch/tasks/shared/LS_root/mounts/clusters/jch122-faster/code/Users/jch122/AzureNormalizePipeline/json_normalizer.py in json_normalizer(device, timePeriod, promptName, data, aggSummary, ignoreInputs, prefix, pandas)
        131     # dealing with no data in output
        132     if output_df.shape == (0,0):
    --> 133         raise Exception("No data in output dataframe. Check inputs.")
        134     else:
        135         print(">>> The shape of the output dataframe is: ", output_df.shape)


    Exception: No data in output dataframe. Check inputs.


The custom exception was added for when a the data outputs a dataframe shape of (0,0). If the the entire dataframe has no data, most likely the inputs are incorrect. In the case above, prompt[1] was from the <code>fitbit</code> <code>current</code> list, so the promptname  <code>'covid_fitbit_steps_intraday_backfill'</code> could not be found in the appropriate keyword-argument list to run <code>json_normalize</code>.

However, there are times when the data has missing rows for specific datasets and can handle a some missing data. See below:


```python
json_normalizer("fitbit", "historical", "covid_fitbit_heart_rate_intraday_backfill", data=df, pandas=True)
```

    >>> The shape of the output dataframe is:  (217540, 9)
    >>> The number of rows that did not have data are: 11


    Do you want see a sample? [y/n] y


            hRZ-max  hRZ-min      hRZ-name  hRZ-minutes  hRZ-caloriesOut  \
    111964       74       30  Out of Range        444.0        389.66028   
    168999      220      125          Peak          0.0          0.00000   
    6224         74       30  Out of Range        444.0        389.66028   
    193240       74       30  Out of Range        827.0        696.29168   
    74738       126      104        Cardio         22.0        113.38328   
    ...         ...      ...           ...          ...              ...   
    179296       74       30  Out of Range        364.0        302.07828   
    131079      220      135          Peak          0.0          0.00000   
    46946       126      104        Cardio         16.0         66.68664   
    214120       74       30  Out of Range        208.0        167.83896   
    108782      126      104        Cardio          1.0          1.19940   
    
           captured_at.dateTime activities-heart.value.restingHeartRate  \
    111964           2020-05-01                                      70   
    168999           2020-07-05                                      68   
    6224             2020-05-01                                      70   
    193240           2020-03-07                                      70   
    74738            2019-10-09                                      69   
    ...                     ...                                     ...   
    179296           2020-08-05                                      74   
    131079           2020-11-24                                      63   
    46946            2019-08-26                                      73   
    214120           2020-07-24                                      74   
    108782           2020-02-23                                      69   
    
            uniqueRowID  participant_id  
    111964      1769065            9032  
    168999      2370051            9032  
    6224        3558015            9032  
    193240      2687980            9032  
    74738       1461419            9032  
    ...             ...             ...  
    179296      2519337            9032  
    131079      3730999            9032  
    46946       1187858            9032  
    214120      2988542            9032  
    108782      1737237            9032  
    
    [2175400 rows x 9 columns]


    Do you want to upload? [y/n] n


### json_normalizer, Demo 3: What is aggSummary?

For some promptNames, especially the Garmin promptNames, there is a aggregated summary that encasulates the entire observation period, and a nested summary that captures more granular observation periods. To illustrate this, the json structure of the promptName <code>covid_garmin_activity</code> for the 'current' <code>timePeriod</code> is shown below. <code>Summary</code> is highlighted in green and <code>aggregatedSummary</code> is highlighted in purple.

<center><img src="./50_assets/summaryAggSummary_compare.png" alt="Comparision of Summary and AggSummary" style="width: 600px;"/></center>

In this dataset, an output for a single line would look like this:

#### <code>aggregatedSummary</code> of covid_garmin_activity:


```python
import pandas as pd
import numpy as np
from pandas import json_normalize
import json

promptValue = "covid_garmin_activity"
chars = '"'
subsetForTransformation = df[(df.prompt_name == promptValue)]

pd_current_json_value = pd.DataFrame(json_normalize(data=json.loads(subsetForTransformation["json_value"].iloc[93].replace("\\","").strip(chars)),))

pd_current_json_value
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>activities</th>
      <th>captured_at</th>
      <th>aggregatedSummary.durationInSeconds</th>
      <th>aggregatedSummary.averageHeartRateInBeatsPerMinute</th>
      <th>aggregatedSummary.distanceInMeters</th>
      <th>aggregatedSummary.steps</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>[{'summaryId': '5133495146-detail', 'summary':...</td>
      <td>2020-06-23</td>
      <td>2402</td>
      <td>135</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



#### <code>Summary</code> of covid_garmin_activity:


```python
pd_current_json_value = pd.DataFrame(json_normalize(
                                        data=json.loads(subsetForTransformation["json_value"].iloc[93].replace("\\","").strip(chars)),
                                        record_path =  "activities", meta =  "captured_at",))
pd_current_json_value
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>summaryId</th>
      <th>samples</th>
      <th>laps</th>
      <th>summary.durationInSeconds</th>
      <th>summary.startTimeInSeconds</th>
      <th>summary.startTimeOffsetInSeconds</th>
      <th>summary.activityType</th>
      <th>summary.averageHeartRateInBeatsPerMinute</th>
      <th>summary.activeKilocalories</th>
      <th>summary.deviceName</th>
      <th>summary.maxHeartRateInBeatsPerMinute</th>
      <th>captured_at</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5133495146-detail</td>
      <td>[{'startTimeInSeconds': 1592924840, 'elevation...</td>
      <td>[{'startTimeInSeconds': 1592924840}]</td>
      <td>2402</td>
      <td>1592924840</td>
      <td>3600</td>
      <td>INDOOR_CARDIO</td>
      <td>135</td>
      <td>387</td>
      <td>vivoactive3</td>
      <td>175</td>
      <td>2020-06-23</td>
    </tr>
  </tbody>
</table>
</div>



The reason that this option exists is that for some promptNames, it makes more sense to use the aggregatedSummary compared to looking at the underlying data. An example of this is if you wanted the step count of an entire day.

### json_normalizer, Demo 4: Ignoring Inputs

If a user wants to iterate over a list prompt names, one may want to turn off the [y/n] user inputs. To do so, set <code>ignoreInputs</code> to <code>True</code>. Normally, one would still want to set the desired prefix of the filepath. Thus, input the desired prefix into <code>prefix</code>.

This is demonstrated below on all the promptNames in Garmin in the <code>current</code> time period (Note: promptName <code>covid_garmin_daily</code>  may take a while to run).


```python
prompt = whichPromptNames("garmin", "current")
#print("\n>>> garmin, current:", prompt)
for pmt in prompt:
    print(f"\n>>>The prompt name is: {pmt}")
    json_normalizer("garmin", "current", pmt, data=df, pandas=True, ignoreInputs=True, prefix="demo_2")
```

    
    >>>The prompt name is: covid_garmin_activity
    >>> The shape of the output dataframe is:  (143, 28)
    >>> The number of rows that did not have data are: 0
    garmin current covid_garmin_activity aggSummary = False
    ./output_csv/demo_2-covid_garmin_activity.csv
    Uploading an estimated of 1 files
    Uploading ./output_csv/demo_2-covid_garmin_activity.csv
    Uploaded ./output_csv/demo_2-covid_garmin_activity.csv, 1 files out of an estimated total of 1
    Uploaded 1 files
    >>> Upload Successful!
    
    >>>The prompt name is: covid_garmin_daily
    >>> The shape of the output dataframe is:  (4661, 35)
    >>> The number of rows that did not have data are: 0
    garmin current covid_garmin_daily aggSummary = False
    ./output_csv/demo_2-covid_garmin_daily.csv
    Uploading an estimated of 1 files
    Uploading ./output_csv/demo_2-covid_garmin_daily.csv
    Uploaded ./output_csv/demo_2-covid_garmin_daily.csv, 1 files out of an estimated total of 1
    Uploaded 1 files
    >>> Upload Successful!
    
    >>>The prompt name is: covid_garmin_epoch
    >>> The shape of the output dataframe is:  (18431, 16)
    >>> The number of rows that did not have data are: 0
    garmin current covid_garmin_epoch aggSummary = False
    ./output_csv/demo_2-covid_garmin_epoch.csv
    Uploading an estimated of 1 files
    Uploading ./output_csv/demo_2-covid_garmin_epoch.csv
    Uploaded ./output_csv/demo_2-covid_garmin_epoch.csv, 1 files out of an estimated total of 1
    Uploaded 1 files
    >>> Upload Successful!
    
    >>>The prompt name is: covid_garmin_sleep
    >>> The shape of the output dataframe is:  (2349, 19)
    >>> The number of rows that did not have data are: 0
    garmin current covid_garmin_sleep aggSummary = False
    ./output_csv/demo_2-covid_garmin_sleep.csv
    Uploading an estimated of 1 files
    Uploading ./output_csv/demo_2-covid_garmin_sleep.csv
    Uploaded ./output_csv/demo_2-covid_garmin_sleep.csv, 1 files out of an estimated total of 1
    Uploaded 1 files
    >>> Upload Successful!


<center><img src="./50_assets/listOfGarminDatasets.png" alt="Comparision of Summary and AggSummary" style="width: 600px;"/></center>

### json_normalizer, Demo 5: Azure Dataset versus Pandas DataFrame

Although <code>json_normalizer</code> can use Azure Datasets, it is advised to use a pandas dataframe instead because the time it takes to run using an Azure Dataset is much greater than a pandas dataframe. See below:


```python
import time

start = time.time()

json_normalizer("fitbit", "historical", "covid_fitbit_steps_intraday_backfill", data=dataset,  ignoreInputs=True, prefix="demo_3", pandas=False)

end = time.time()

print(f"Time ellapsed: {end - start:.5f} seconds.")
```

    >>> The shape of the output dataframe is:  (58400, 4)
    >>> The number of rows that did not have data are: 0
    fitbit historical covid_fitbit_steps_intraday_backfill aggSummary = False
    ./output_csv/demo_3-covid_fitbit_steps_intraday_backfill.csv
    Uploading an estimated of 1 files
    Uploading ./output_csv/demo_3-covid_fitbit_steps_intraday_backfill.csv
    Uploaded ./output_csv/demo_3-covid_fitbit_steps_intraday_backfill.csv, 1 files out of an estimated total of 1
    Uploaded 1 files
    >>> Upload Successful!
    Time ellapsed: 40.05135 seconds.



```python
import time

start = time.time()

json_normalizer("fitbit", "historical", "covid_fitbit_steps_intraday_backfill", data=df,  ignoreInputs=True, prefix="demo_3", pandas=True)

end = time.time()

print(f"Time ellapsed: {end - start:.5f} seconds.")
```

    >>> The shape of the output dataframe is:  (58400, 4)
    >>> The number of rows that did not have data are: 0
    fitbit historical covid_fitbit_steps_intraday_backfill aggSummary = False
    ./output_csv/demo_3-covid_fitbit_steps_intraday_backfill.csv
    Uploading an estimated of 1 files
    Uploading ./output_csv/demo_3-covid_fitbit_steps_intraday_backfill.csv
    Uploaded ./output_csv/demo_3-covid_fitbit_steps_intraday_backfill.csv, 1 files out of an estimated total of 1
    Uploaded 1 files
    >>> Upload Successful!
    Time ellapsed: 3.36645 seconds.



```python

```
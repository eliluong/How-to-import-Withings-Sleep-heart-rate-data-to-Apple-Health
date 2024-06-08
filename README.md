# How-to-import-Withings-Sleep-heart-rate-data-to-Apple-Health
I purchased the Withings Sleep Tracking Mat to track my sleep and my heart rate because wearing a watch at night was not comfortable. The Withings app collects sleep stage data, heart rate, and breathing disturbances. While sleep data is written to Apple Health, the heart rate data is not. Here is a method to get the heart rate data out of Withings and import it into Apple Health via Shortcuts. There are ways to automate it further, including using the Withings API, but this is it for now.

## 1. Export data from Withings
Read these [instructions](https://support.withings.com/hc/en-us/articles/201491377-Withings-App-Online-Dashboard-Exporting-my-data) on how to export your Withings data. Withings will send you an email with a link to download your data. In the Zip file, `raw_bed_hr.csv` contains the heart rate data.

Open this file in a text editor. The first row is the header row. The subsequent rows are generated from the Sleep Tracking Mat, but may also contain data from other sources. In my CSV file, the heart rate values were duplicated for some reason, so I took what I needed, and put it into a separate CSV file.

The structure of the data is as follows. `start` is the time this row of data starts. It is in `+02:00` timezone. `duration` and `value` are both arrays of the same length. I believe each element in `duration` represents seconds, and it signifies either the average heart rate over 60 seconds, or how long until the next reading. In any case, the Sleep Tracking Mat is recording heart rate every 60 seconds into the `value` variable.
```csv
start,duration,value
2024-06-07T09:22:00+02:00,"[60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60]","[65,65,63,62,67,64,63,63,61,64,66,63,65,66,65,64,65,65,64,66,65,66,66,67,66,67,66,66,68,66,65,66,67,66,66,68,68,68,67,68,67,67,67,68,67,68,68,67,68,68,68,68,68,67,68,68,67,68,69,68,69,69,69,69,69,69,69,67,69,72,73,73,75,75,73,76,74,76,76,73,73,75,73,72,70,68,68,69,68,68,69,69,66,65,67,65,66,66,67,67,67,66,67,67,66,67,66,66,66,66,65,66,66,66,65,65,66,66,65,64,64,65,65,65,65,65,65,65,65,64,64,65,66,65,65,66,65,66,64,61,63,63,64,64,65,64,71,73,72,71,71,73,72,71,64,69,63,65,67,66,65,66,64,67,68,66,69,67,66,66,71,70,68,71,69,62,64,62,61,66]"
2024-06-07T12:22:00+02:00,"[60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60]","[62,64,63,61,61,62,59,60,59,60,60,57,55,57,57,58,57,58,58,58,59,61,55,56,56,56,56,57,56,56,55,55,56,56,56,56,56,56,56,56,56,56,56,55,57,58,56,57,58,58,58,60,56,57,55,58,58,60,64,59,59,58,69,62,60,60,59,59,62,61,57,58,60,59,60,60,59,56,62,60,63,65,65,66,64,65,65,65,66,66,66,66,65,66,66,66,67,64,62,61,61,62,61,60,60,59,60,61,60,60,60,59,61,63,63,65,65,64,66,64,66,65,64,65,66,67,71,72,70,71,72,71,64,65,69,68,71,70,68,66,73,73,73,73,70,71]"
2024-06-07T14:48:00+02:00,"[60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60,60]","[72,66,69,66,68,66,65,67,63,63,60,65,58,58,57,60,57,57,57,56,58,57,57,60,58,59,57,54,58,58,58,58,58,57,58,58,57,58,57,58,57,57,58,56,57,58,57,58,57,58,59,57,57,57,57,57,57,57,57,57,58,58,59,58,57,56,57,57,57,57,57,59,57,57,58,58,60,57,60,59,55,58,61,62,60,60,57,60,62,63,61,61,57,57,58,58,57,58,57,57,57,54,58,60,58,59,58,59,56,59,58,58,58,58,57,59,58,58,59,58,58,58,57,59,58,59,57,58,58,58,60,58,59,58,60,56,55,56,53,57,54,56,56,56,54,55,57,60,59,56,58,58,58,59,60,60,60,59,60,60,62,62,61,58,64,66,64,66,65,64,66,70,73,75,77,71,70,65,65,64]"
2024-06-07T17:48:00+02:00,"[60,60,60,60,60,60,60,60,60,60]","[67,69,66,66,65,70,61,68,66,68]"
```

## 2. Process CSV into ingestible format for Apple Health
This Python script will take the CSV file, correct the time to my timezone and an acceptable format for importing into Apple Health, and parse the `value` column. The last part of the script iterates through the dataframe and outputs it as `datetime,value` format.

You will need to adjust the timezone to where you are. I am not sure what will happen when DST starts/ends, but I will find that out later.

Since the Sleep Tracking Mat is producing a heart rate value every minute, I chose to import heart rate every three minutes to avoid cluttering Apple Health with excessive data. You can adjust this as needed.
```python
import pandas as pd
import json
import ast

df = pd.read_csv('withings.csv')
df.drop('duration', axis=1, inplace=True)
df['start'] = pd.to_datetime(df['start'])
df['start_pst'] = df['start'].dt.tz_convert('America/Los_Angeles').dt.strftime('%Y-%m-%d %H:%M:%S')
df['value'] = df['value'].apply(ast.literal_eval) # convert string to array

output_data = []

for i, row in df.iterrows():
    counter = 0
    datetime = pd.to_datetime(row['start_pst'])
    for value in row['value']:
        new_datetime = datetime + pd.Timedelta(minutes=counter)
        if counter % 3 == 0:
            print(new_datetime, value)
            output_data.append([new_datetime, value])
        counter += 1
output_df = pd.DataFrame(output_data)
output_df.to_csv('output.csv', header=False, index=False)
```
## 3. Create an API to access the data
I will let Apple Shortcuts access an API to get the data for import. You can do this via whatever method works best. I used a PHP script as it was the easiest to set up.
```php
<?php
header('Content-Type: application/json');
$csvFilePath = 'output.csv';

$data = array();
if (($handle = fopen($csvFilePath, "r")) !== FALSE) {
    while (($row = fgetcsv($handle, 1000, ",")) !== FALSE) {
        $data[] = array(
            'timestamp' => $row[0],
            'value' => intval($row[1])
        );
    }
    fclose($handle);
}

echo json_encode($data);
?>
```

## 4. Create Shortcuts shortcut to import the data
Create the shortcut as follows. You should add `Log Health Sample` first and run the shortcut in order to get Apple Health to allow Shortcuts to write data. Be sure to point the `Value` and `Date` variables in `Log Health Sample` component to the correct `Get Value` components.

This shortcut gets data from the API, reads it as a dictionary, iterates over each item, then writes each item to Apple Health. The data should then be immediately viewable in Apple Health.

![image](https://github.com/eliluong/How-to-import-Withings-Sleep-heart-rate-data-to-Apple-Health/assets/15806938/df695957-ce27-4edd-817f-2e9460f3a16d)

That's it. The drawbacks are having to manually export Withings data each time, keeping track of what data or what days have been imported.

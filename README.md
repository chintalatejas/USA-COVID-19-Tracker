# USA COVID-19 Tracker

## Introduction
This notebook utilized the Structured Streaming Spark SQL engine to create a weekly tracker for monitoring COVID-19 cases and deaths across the different USA states and counties, with the seamless integration of sliding and tumbling window operations to ensure real-time and comprehensive data analysis, enhancing the precision and granularity of our insights into the evolving pandemic landscape at the regional level. 

## Dataset

The data was taken from the Kaggle dataset: [New York Times - COVID -19 Dataset](https://www.kaggle.com/datasets/kalilurrahman/new-york-times-covid19-dataset/data) This Kaggle dataset has 5 CSV files, for this project, "us_counties.csv" file is only used. This csv file has 6 columns:
- date - Recorded date ranging from 20-01-2020 to 12-05-2022
- county - The name fo the county
- state - the name of the state for that particular county
- fips - county fips or the unique 5-digit code for that county
- cases - number of Covid-19 cases for that particular country on that date
- deaths - number of Covid-19 deaths for that particular county on that date

## Methodology

The goal is to obtain the count of COVID-19 cases and deaths for each state on a day-to-day basis, hence an aggregation is performed on the data frame. To enhance the real-time aspect of the analysis, the main CSV file is
partitioned into multiple subsequent CSV files. Each file represents a streaming input source and contains data
on COVID-19 cases and deaths for all counties in the USA, organized by date. information. These subsequent CSV files
(a total of 1690) present in this directory portray the streaming input source needed for Structured Streaming.

An initialization involves leveraging the readStream functionality, that allows each file to process as a stream, Given the substantial
volume of data, we configure the "maxFilesPerTrigger" parameter to 10. This setting ensures that each micro-batch
will encompass 10 files, effectively managing the data influx. 





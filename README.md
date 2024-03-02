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
will encompass 10 files, effectively managing the data influx. The sink is defined  which is responsible for storing the output in external systems using the writeStream method. The sink’s
output mode is set to “complete,” ensuring the entire result is written to the in-memory table at once.

A modification is made to the data aggregration where the data is grouped solely based on the state column. This adjustment enables us to derive comprehensive insights into the cumulative cases and deaths for each state.

### Window Operations

For a more in-depth analysis and trend identification, leading COVID-19 dashboards often rely on statistics computed
over specific time periods. To facilitate this deeper analysis, we leverage window operations within Spark Streaming.
The different types of window aggregations used here are:

• Tumbling Window - Fixed-sized and non-overlapping windows, where each element is associated with a single
window. In our case, we set a window duration of 14 days to capture data within distinct 14-day intervals.
• Sliding Window - Overlapping windows, necessitating the specification of a sliding offset and interval. Similar
to the tumbling window, we set a window duration of 14 days. However, a sliding offset of 7 days is introduced
to define overlapping intervals.

## Results

Tables 1 and 2 illustrate the initial aggregation process, focusing on obtaining the total number of cases and deaths per
state for the first 5 states in alphabetical order.

#### Table 1. Total number of Cases and Deaths per State (Batch 25)

| State     | Total Cases | Total Deaths |
|-----------|-------------|--------------|
| Alabama   | 10,030,367  | 191,158      |
| Alaska    | 1,098,655   | 53,56        |
| Arizona   | 16,211,704  | 327,778      |
| Arkansas  | 6,211,616   | 101,364      |
| California| 67,582,505  | 1,067,103    |

#### Table 2. Total number of Cases and Deaths per State (Batch 100)

| State     | Total Cases | Total Deaths |
|-----------|-------------|--------------|
| Alabama   | 67,827,958  | 1,222,514    |
| Alaska    | 7,498,620   | 35,300       |
| Arizona   | 10,651,1682 | 2,140,690    |
| Arkansas  | 4,188,5584  | 684,266      |
| California| 44,251,4630 | 6,827,495    |

Tables 3 and 4 showcase the output of the tumbling window operation. This provides a comprehensive
view of the number of Covid-19 cases and deaths within distinct 14-day periods. Notably, the fixed data received in
each window, synchronized with the micro-batch updates, enables nuanced trend analysis, offering insights into the
evolving patterns of the pandemic.

#### Table 3. Tumbling Window: Number of Cases and Deaths per 14 day period for Ohio (Batch 25)

| Start                | End                  | State | Total Cases | Total Deaths |
|----------------------|----------------------|-------|-------------|--------------|
| 2020-03-05 00:00:00 | 2020-03-19 00:00:00 | Ohio  | 130         | 0            |
| 2020-03-19 00:00:00 | 2020-04-02 00:00:00 | Ohio  | 4,049       | 86           |
| 2020-04-02 00:00:00 | 2020-04-16 00:00:00 | Ohio  | 24,281      | 916          |
| 2020-04-16 00:00:00 | 2020-04-30 00:00:00 | Ohio  | 39,116      | 1,698        |
| 2020-04-30 00:00:00 | 2020-05-14 00:00:00 | Ohio  | 82,971      | 4,516        |

#### Table 4. Tumbling Window: Number of Cases and Deaths per 14 day period for Ohio (Batch 100)

| Start                | End                  | State | Total Cases | Total Deaths |
|----------------------|----------------------|-------|-------------|--------------|
| 2020-03-05 00:00:00 | 2020-03-19 00:00:00 | Ohio  | 298         | 0            |
| 2020-03-19 00:00:00 | 2020-04-02 00:00:00 | Ohio  | 14,364      | 284          |
| 2020-04-02 00:00:00 | 2020-04-16 00:00:00 | Ohio  | 74,673      | 2,798        |
| 2020-04-16 00:00:00 | 2020-04-30 00:00:00 | Ohio  | 191,916     | 8,679        |
| 2020-04-30 00:00:00 | 2020-05-14 00:00:00 | Ohio  | 307,739     | 16,977       |

Tables 5 and 6 introduce the sliding window operation, a distinctive feature where the computation spans a 14-day
interval, but due to a sliding offset of 7 days, some data spans across multiple windows. An illustrative example is
found in Table 6, where the interval starting on “2020-02-27 00:00:00” accumulates 10 cases. The subsequent interval,
commencing on “2020-03-05 00:00:00,” is just 7 days apart from the start of the previous cycle, effectively incorporating
data from the prior window showcasing the overlapping window nature.

#### Table 5. Sliding Window: Number of Cases and Deaths for Ohio per 14 day period with 7 day slide interval (Batch 25)

| Start                | End                  | State | Total Cases | Total Deaths |
|----------------------|----------------------|-------|-------------|--------------|
| 2020-02-27 00:00:00 | 2020-03-12 00:00:00 | Ohio  | 6           | 0            |
| 2020-03-05 00:00:00 | 2020-03-19 00:00:00 | Ohio  | 146         | 0            |
| 2020-03-12 00:00:00 | 2020-03-26 00:00:00 | Ohio  | 1,291       | 17           |
| 2020-03-19 00:00:00 | 2020-04-02 00:00:00 | Ohio  | 6,689       | 137          |
| 2020-03-26 00:00:00 | 2020-04-09 00:00:00 | Ohio  | 14,031      | 381          |

#### Table 6. Sliding Window: Number of Cases and Deaths for Ohio per 14 day period with 7 day slide interval (Batch 100)

| Start                | End                  | State | Total Cases | Total Deaths |
|----------------------|----------------------|-------|-------------|--------------|
| 2020-02-27 00:00:00 | 2020-03-12 00:00:00 | Ohio  | 10          | 0            |
| 2020-03-05 00:00:00 | 2020-03-19 00:00:00 | Ohio  | 298         | 0            |
| 2020-03-12 00:00:00 | 2020-03-26 00:00:00 | Ohio  | 2,890       | 32           |
| 2020-03-19 00:00:00 | 2020-04-02 00:00:00 | Ohio  | 14,364      | 284          |
| 2020-03-26 00:00:00 | 2020-04-09 00:00:00 | Ohio  | 40,141      | 1,147        |





# Fitness Data Insights: Leveraging Analytics for Health and Wellness Optimization

![goran-ivos-iOykDIkZLQw-unsplash](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/c1b96f6a-c193-45b4-976c-a9609fe987e8)

### Introduction
---
Bellabeat is a leading technology company specializing in health and wellness products designed for women. Renowned for their innovative smart devices and wearable technology, Bellabeat integrates technology seamlessly with fashion to empower women in tracking and managing various aspects of their health and well-being. Their products offer features such as activity tracking, sleep monitoring, stress management, and menstrual cycle tracking, all with a stylish design focus.

![image](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/4e5f040d-6d31-42df-8afe-9d4ffbca1b07)

### Main Task
---
The main task here is to analyze smart device usage data to gain insights into how consumers utilize non-Bellabeat smart devices. This analysis aims to answer key questions regarding trends in smart device usage, how these trends may apply to Bellabeat customers, and how they could potentially influence Bellabeat's marketing strategy.

### Ask Phase
---
1. What are some trends in smart device usage?
2. How could these trends apply to Bellabeat customers?
3. How could these trends help influence Bellabeat's marketing strategy?

### Prepare
---
The dataset for this analysis is publicly available on Kaggle and consists of data collected from a survey conducted via Amazon Mechanical Turk between March 12, 2016, and May 12, 2016. Thirty qualified Fitbit users provided consent to share their personal tracking data, including detailed information about physical activity, heart rate, and sleep patterns recorded on a minute-by-minute basis. It's worth noting that variations in data output exist due to the different types of Fitbit trackers used by participants and their unique tracking behaviors and preferences.

### Process
---
A variety of tools were utilized for this analysis, including Excel, SQL, BigQuery, Tableau, and Looker Studio. The dataset was downloaded and organized in a folder with proper file-naming conventions. Data cleaning and manipulation processes were documented using Kaggle notebook, ensuring that the data was ready for analysis. Duplicate data was identified and corrected using Excel and SQL, with data types adjusted as needed. For instance, numerical data were formatted as numbers, and activity dates were converted to date format using Tableau.

### Analyze
---
A dedicated dataset named "Bellabeats" was created in BigQuery, with tables created and data uploaded in CSV format. Users were categorized based on their app engagement frequency, and visualizations were generated using Tableau, Looker, and PowerPoint to extract meaningful insights from the data.


### Data Cleaning and Analysis in  Google Big Query
---
```sql
-- Count number of rows in main table - daily_activity
SELECT COUNT(*)
FROM `bellabeats97440.fitness_data.daily_activity`;

-- Checking for number of unique ids in each of the tables provided for this task
SELECT COUNT(DISTINCT Id)
FROM `bellabeats97440.fitness_data.daily_activity`;

SELECT COUNT(DISTINCT Id)
FROM `bellabeats97440.fitness_data.sleep_day`;

SELECT COUNT(DISTINCT Id)
FROM `bellabeats97440.fitness_data.daily_intensities`;

SELECT COUNT(DISTINCT Id)
FROM `bellabeats97440.fitness_data.weight_log`;

SELECT COUNT(DISTINCT Id)
FROM `bellabeats97440.fitness_data.heartrate_merged`;

-- Data from weight_log and heart_rate tables are insufficient and thereby excluded from our analysis

-- Checking for null values
SELECT da.Id, da.ActivityDate, ActivityDay
FROM `bellabeats97440.fitness_data.daily_activity` da
LEFT JOIN `bellabeats97440.fitness_data.daily_calories` dc
ON da.Id = dc.Id AND da.ActivityDate = dc.ActivityDay
WHERE dc.ActivityDay IS NULL OR da.ActivityDate IS NULL;

SELECT da.Id, da.ActivityDate, ActivityDay
FROM `bellabeats97440.fitness_data.daily_activity` da
LEFT JOIN `bellabeats97440.fitness_data.daily_calories` dc
ON da.Id = dc.Id AND da.ActivityDate = dc.ActivityDay;

-- Checking for duplicates in daily_activity table
SELECT Id, ActivityDate, SedentaryMinutes, TotalDistance, TotalSteps, COUNT(*) AS count
FROM `bellabeats97440.fitness_data.daily_activity`
GROUP BY Id, ActivityDate, SedentaryMinutes, TotalDistance, TotalSteps
HAVING COUNT(Id) > 1;

-- Checking for duplicates in sleep_day table
SELECT Id, SleepDay, TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed, COUNT(*) AS count
FROM `bellabeats97440.fitness_data.sleep_day`
GROUP BY Id, SleepDay, TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed
HAVING COUNT(Id) > 1;

-- Checking for duplicates in weight_log table
SELECT Id, Date, WeightKg, BMI, LogId, COUNT(*) AS count
FROM `bellabeats97440.fitness_data.weight_log`
GROUP BY Id, Date, WeightKg, BMI, LogId
HAVING COUNT(Id) > 1;

-- Checking for start and end dates
SELECT MIN(SleepDay), MAX(SleepDay)
FROM `bellabeats97440.fitness_data.sleep_day`;

SELECT MIN(ActivityDate), MAX(ActivityDate)
FROM `bellabeats97440.fitness_data.daily_activity`;

SELECT MIN(Date), MAX(Date)
FROM `bellabeats97440.fitness_data.weight_log`;

-- Check column length
SELECT Id
FROM `bellabeats97440.fitness_data.daily_activity`
WHERE LENGTH(CAST(Id AS STRING)) > 10 OR LENGTH(CAST(Id AS STRING)) < 10;

SELECT Id
FROM `bellabeats97440.fitness_data.sleep_day`
WHERE LENGTH(CAST(Id AS STRING)) > 10 OR LENGTH(CAST(Id AS STRING)) < 10;

SELECT Id
FROM `bellabeats97440.fitness_data.weight_log`
WHERE LENGTH(CAST(Id AS STRING)) > 10 OR LENGTH(CAST(Id AS STRING)) < 10;

-- Same id length and start to end date
SELECT Id, ActivityDate, SedentaryMinutes, Calories 
FROM `bellabeats97440.fitness_data.daily_activity`
GROUP BY Id, ActivityDate, SedentaryMinutes, Calories
ORDER BY ActivityDate DESC;


SELECT Id, ActivityDate, SedentaryMinutes, Calories 
FROM `bellabeats97440.fitness_data.daily_activity`
WHERE Calories = 0 OR SedentaryMinutes = 0
ORDER BY ActivityDate DESC;

SELECT Id, ActivityDate, SedentaryMinutes, Calories 
FROM `bellabeats97440.fitness_data.daily_activity`
WHERE Calories >= 1 AND SedentaryMinutes >= 1
ORDER BY ActivityDate DESC;

SELECT Id, ActivityDate, ROUND(AVG(Calories)) AS Avg_Calories
FROM `bellabeats97440.fitness_data.daily_activity`
GROUP BY Id, ActivityDate;
```

### Categorization of Users
---
Participants who used the app for more than 25 days out of 31 days were categorized as 'Frequent User' while between 15 and 24, 'Moderate User' and the last group 'Light User' logged in to the app less than 15 days during the entire period.
```sql
-- Classification of different categories of users
SELECT 
    Id, 
    Count(id) AS total_times_logged,
    CASE
        WHEN Count(id) BETWEEN 25 AND 31 THEN 'Frequent User'
        WHEN Count(id) BETWEEN 15 AND 24 THEN 'Moderate User'
        WHEN Count(id) BETWEEN 0 AND 14 THEN 'Light User'
    END AS app_user_category
FROM 
    `bellabeats97440.fitness_data.daily_activity`
GROUP BY 
    id;
```

![Categories of user](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/472c15e8-5e1e-4be6-b454-c9df3fb96165)

![User by Category Looker](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/1d2a0024-ef02-41aa-a0e1-e63860464a53)

Average intensity of daily activity was also calculated for each participant using SQL. 
---
```sql
-- Calculate average intensity of daily activity for each participant
SELECT 
    Id, 
    Round(Avg(SedentaryMinutes)) AS sedentary_mins, 
    Round(Avg(LightlyActiveMinutes)) AS lightlyActive_mins, 
    Round(Avg(FairlyActiveMinutes)) AS fairlyActive_mins, 
    Round(Avg(VeryActiveMinutes)) AS veryActive_mins
FROM 
    `bellabeats97440.fitness_data.daily_intensities`
GROUP BY 
    id;
```
![Average daily intensity](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/8d8b76d3-160d-4b2e-93f1-d0b644035109)

![image](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/6912c119-d685-4295-9733-40ad6b0da541)

![image](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/593d7752-4d19-4435-b724-f46ede93aae3)

![image](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/d1d92e05-64bd-4186-8a61-d13f9c31421a)

![image](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/e7fe0387-37ef-4e0b-83f5-ad0ee4827928)

```sql
-- Querying the average number of time spent active by Id
SELECT 
    Id, 
    Round(Avg(SedentaryMinutes)) AS sedentary_mins, 
    Round(Avg(LightlyActiveMinutes)) AS lightlyActive_mins, 
    Round(Avg(FairlyActiveMinutes)) AS fairlyActive_mins, 
    Round(Avg(VeryActiveMinutes)) AS veryActive_mins
FROM 
    `bellabeats97440.fitness_data.daily_intensities`
GROUP BY 
    id
ORDER BY 
    sedentary_mins DESC;

-- Same query as immediately above but from the merged daily activity table for a quick check
SELECT 
    Id, 
    Round(Avg(SedentaryMinutes)) AS sedentary_mins, 
    Round(Avg(LightlyActiveMinutes)) AS lightlyActive_mins, 
    Round(Avg(FairlyActiveMinutes)) AS fairlyActive_mins, 
    Round(Avg(VeryActiveMinutes)) AS veryActive_mins
FROM 
    `bellabeats97440.fitness_data.daily_activity`
GROUP BY 
    id
ORDER BY 
    sedentary_mins DESC 
LIMIT 
    10;

-- Aggregate insights: No of days a user's id appeared on the records
SELECT 
    Id, 
    COUNT(Id) AS number_of_times_logged
FROM 
    `bellabeats97440.fitness_data.daily_activity`
GROUP BY 
    Id;

-- Activity intensity vs calories
SELECT 
    Id, 
    Round(Avg(SedentaryMinutes)) AS sedentary_mins, 
    Round(Avg(LightlyActiveMinutes)) AS lightlyActive_mins, 
    Round(Avg(FairlyActiveMinutes)) AS fairlyActive_mins, 
    Round(Avg(VeryActiveMinutes)) AS veryActive_mins, 
    Round(Avg(Calories)) AS calories_burned
FROM 
    `bellabeats97440.fitness_data.daily_activity`
GROUP BY 
    Id;
```

Average calories burned vs Very active minute
---
![image](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/7d473125-f10f-458e-923e-e390ae6c61e3)

```sql
SELECT 
    Id, 
    Round(Avg(SedentaryMinutes)) AS sedentary_mins, 
    Round(Avg(LightlyActiveMinutes)) AS lightlyActive_mins, 
    Round(Avg(FairlyActiveMinutes)) AS fairlyActive_mins, 
    Round(Avg(VeryActiveMinutes)) AS veryActive_mins, 
    Round(Avg(Calories)) AS calories_burned
FROM 
    `bellabeats97440.fitness_data.daily_activity`
GROUP BY 
    Id;

SELECT 
    Id, 
    Sedentary_mins, 
    (lightlyActive_mins + fairlyActive_mins + veryActive_mins) AS Total_ActiveMinutes, 
    Calories_burned
FROM 
    (
        SELECT 
            Id, 
            Round(Avg(SedentaryMinutes)) AS Sedentary_mins, 
            Round(Avg(LightlyActiveMinutes)) AS lightlyActive_mins, 
            Round(Avg(FairlyActiveMinutes)) AS fairlyActive_mins, 
            Round(Avg(VeryActiveMinutes)) AS veryActive_mins, 
            Round(Avg(Calories)) AS Calories_burned
        FROM 
            `bellabeats97440.fitness_data.daily_activity`
        GROUP BY 
            Id
    );

SELECT 
    Id, 
    (SedentaryMinutes) AS sedentary_mins, 
    (LightlyActiveMinutes + FairlyActiveMinutes + VeryActiveMinutes) AS TotalActive_mins, 
    (Calories) AS calories_burned
FROM 
    `bellabeats97440.fitness_data.daily_activity`
ORDER BY 
    Id;

SELECT 
    ActivityDay, 
    Round(Avg(SedentaryMinutes)) AS Avg_Sedentary_mins, 
    Round(Avg(LightlyActiveMinutes)) AS Avg_LightlyActive_mins, 
    Round(Avg(FairlyActiveMinutes)) AS Avg_FairlyActive_mins, 
    Round(Avg(VeryActiveMinutes)) AS Avg_VeryActive_mins
FROM 
    `bellabeats97440.fitness_data.daily_activity2`
GROUP BY 
    ActivityDay
ORDER BY 
    ActivityDay ASC;

-- Max and min number of daily steps
SELECT 
    Max(TotalSteps) AS Max_DailySteps, 
    MIN(TotalSteps) AS Min_DailySteps
FROM 
    `bellabeats97440.fitness_data.daily_activity2`;

-- Average number of daily steps
SELECT 
    ActivityDay, 
    Round(Avg(TotalSteps)) AS Avg_DailySteps
FROM 
    `bellabeats97440.fitness_data.daily_activity2`
GROUP BY 
    ActivityDay ;
```
![image](https://github.com/Melrowze/BigQuery_Looker/assets/44920093/fcdae6b5-9b19-40da-8d4a-231bc5a4b0e8)

### Recommendation
---
Based on the analysis conducted on the Bellabeats fitness data, several recommendations can be made:

1. **Engage with Frequent Users:** Identify and target users who engage with the app frequently (25 to 31 days out of 31 days) as they represent highly active and committed users. Consider offering them exclusive rewards, discounts, or personalized experiences to enhance their loyalty and satisfaction.

2. **Encourage Moderate Users:** Encourage users who engage with the app moderately (15 to 24 days out of 31 days) to increase their activity levels and usage frequency. Implement strategies such as personalized notifications, challenges, or goal-setting features to motivate them to be more active and consistent in using the app.

3. **Re-engage Light Users:** Re-engage users who use the app less frequently (0 to 14 days out of 31 days) by offering incentives, reminders, or content tailored to their interests and preferences. Focus on providing valuable and relevant content to encourage them to incorporate the app into their daily routines.

4. **Improve Activity Intensity Tracking:** Enhance the app's ability to track and analyze users' activity intensity levels, including sedentary, lightly active, fairly active, and very active minutes. Provide users with detailed insights into their activity patterns and progress towards their fitness goals to empower them to make informed decisions about their health and wellness.

5. **Optimize Calorie Burn Monitoring:** Improve the accuracy and reliability of the app's calorie burn monitoring feature by refining the algorithms and data processing methods used. Ensure that users receive accurate information about their calorie expenditure during different activities and exercises to support their fitness and weight management efforts.

6. **Enhance Data Visualization:** Enhance the visualizations and reports generated from the fitness data to make them more intuitive, interactive, and actionable. Utilize advanced data visualization tools and techniques to present complex information in a clear and comprehensible manner, enabling users to interpret and act upon the insights effectively.

7. **Offer Personalized Recommendations:** Leverage the insights derived from the data analysis to offer personalized recommendations and insights to users based on their activity levels, preferences, and goals. Provide tailored workout plans, nutrition advice, and lifestyle tips to help users optimize their health and fitness outcomes.

By implementing these recommendations, Bellabeats can enhance user engagement, satisfaction, and retention, ultimately driving the success and growth of their smart fitness technology platform.

Thank you for your attention.
---

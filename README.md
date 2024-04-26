# Bella beat smart device analysis

### Project Overview 
Bella Beat wants to better understand how consumers use smart health devices in order to optimize their marketing strategies and potentially uncover new product development opportunities. They want to Analyze the Fitbit Fitness Tracker Data to identify patterns in how people utilize their smart devices and then develop high-level marketing strategy recommendations informed by the insights gained from the data analysis

### Data Sources used in project

DailyActivity_merged: This dataset contains each user data from April till May on each day which contains each user	TotalSteps,	TotalDistance,	TrackerDistance, LoggedActivitiesDistance,	VeryActiveDistance,	ModeratelyActiveDistance,	LightActiveDistance,	SedentaryActiveDistance,	VeryActiveMinutes,	FairlyActiveMinutes,	LightlyActiveMinutes,	SedentaryMinutes,	Calories

dailyCalories_merged: This dataset contains calorie measure on each day.

dailyIntensities_merged: This dataset contains intensity measure on each day.

dailySteps_merged: This dataset contains steps measure on each day.

heartrate_seconds_merged: The dataset contains each user's heart rate, recorded every second, from April to May.

hourlyCalories_merged: This dataset Contains each user calorie recorded every hour from April till May.

hourlyIntensities_merged: This dataset Contains each user Intensity of steps, recorded every hour from April till May.

hourlySteps_merged: This dataset Contains each user steps, recorded every hour from April till May.

Sleepday_merged: This dataset contains Sleep each user sleep records from April till may.

### Tools

Excel: Data cleaning And Data validation

SQL: Used to extract the required data to answer analysis questions.

Tableau: Used to Visualize the data to generate insights from it and answer Bella Beat Problem.

### Exploratory Data Analysis (EDA)

- EDA involves exploring the Bella Beat smart device dataset to answer the key questions for Analysis for example:

  - How many users are actively engaging with Bellabeat smart devices on a daily basis?
  - What is the average duration of daily usage for Bellabeat smart devices among users?
  - What are the most frequently used features of Bellabeat smart devices among users?
  - How does user engagement with Bellabeat smart devices change over time (e.g., daily, weekly, monthly trends)?
  - What are the peak usage hours for Bellabeat smart devices throughout the day?
  - How does user engagement differ between different types of Bellabeat products (e.g., activity trackers, sleep trackers)?
  - What are the primary reasons for user churn or discontinuation of Bellabeat smart device usage?

### Code Snippet

```sql
SELECT 
      DATE(dailyactivity_merged.date_1) AS activity_date, 
      COUNT(DISTINCT dailyactivity_merged.id) AS daily_active_users
FROM dailyactivity_merged INNER JOIN dailycalories_merged 
     ON dailyactivity_merged.index= dailycalories_merged.index
	 INNER JOIN dailyintensities_merged ON dailycalories_merged.index
	 = dailyintensities_merged.index INNER JOIN dailysteps_merged ON
	 dailyintensities_merged.index =dailysteps_merged.index
GROUP BY  activity_date; 

-- average duration of daily usage for Bellabeat smart devices 
SELECT 
    id, 
    activity_date,
    AVG(avg_usage_minutes) AS avg_usage_minutes_1
FROM (
    SELECT 
        id, 
        DATE(date_1) AS activity_date,
        ROUND(EXTRACT(EPOCH FROM (MAX(time_1) - MIN(time_1))) / 60) AS avg_usage_minutes
    FROM 
        Public.hourlycalories_merged
    GROUP BY 
        id, 
        activity_date

    UNION

    SELECT
        id,
        DATE(date_1) AS activity_date,
        ROUND(EXTRACT(EPOCH FROM (MAX(time_1) - MIN(time_1))) / 60) AS avg_usage_minutes
    FROM
        Public."hourlyintensities_merged"
    GROUP BY
        id,
        activity_date

    UNION

    SELECT
        id,
        DATE(date_1) AS activity_date,
        ROUND(EXTRACT(EPOCH FROM (MAX(time_1) - MIN(time_1))) / 60) AS avg_usage_minutes
    FROM
        Public."hourlysteps_merged"
    GROUP BY
        id,
        activity_date
) AS subquery
GROUP BY 
    id, 
    activity_date;






-- Frequent feature usage
WITH feature_usage AS (
    SELECT 
        SUM(subquery.calories) AS Calorie_count,
        SUM(subquery.steptotal) AS Steps_count, 
        SUM(subquery.totalintensity) AS intensity_count,
        EXTRACT(Month from subquery.date_1) AS activity_day,
	    SUM(subquery.value) AS value_count,
        subquery.id AS id
    FROM (
        SELECT 
            hourlycalories_merged.id, 
            hourlycalories_merged.calories,
            hourlysteps_merged.steptotal, 
            hourlyintensities_merged.totalintensity,
            hourlycalories_merged.date_1,heartrate_seconds_merged.value
        FROM
            Public.hourlycalories_merged
            INNER JOIN Public.hourlyintensities_merged ON hourlycalories_merged.Index = hourlyintensities_merged.Index
            INNER JOIN Public.hourlysteps_merged ON hourlyintensities_merged.Index = hourlysteps_merged.Index
		    INNER JOIN public.heartrate_seconds_merged ON hourlysteps_merged.Index = heartrate_seconds_merged.Index  ) AS subquery 
    GROUP BY subquery.id, subquery.date_1
)

 SELECT 
        ROUND(AVG(feature_usage.Calorie_count)) AS Calorie,
       ROUND(AVG(feature_usage.Steps_count)) AS steps,
        ROUND(AVG(feature_usage.intensity_count)) AS intensity,
		ROUND(AVG(feature_usage.value_count)) AS value_heart,
        feature_usage.activity_day
FROM 
    feature_usage
GROUP BY 
     feature_usage.id, feature_usage.activity_day
ORDER BY  feature_usage.activity_day

		 
-- DAILY ENGAGEMENT OF USERS WITH SMART DEVICES
SELECT
extract(day from dailyactivity_merged.date_1) AS activity_day, 
  COUNT(DISTINCT dailyactivity_merged.id) AS weekly_active_users
FROM Public."dailyactivity_merged" INNER JOIN Public."dailycalories_merged"
     ON dailyactivity_merged.index= dailycalories_merged.index
	 INNER JOIN Public."dailyintensities_merged" ON dailycalories_merged.index
	 = dailyintensities_merged.index INNER JOIN Public."dailysteps_merged" ON
	 dailyintensities_merged.index =dailysteps_merged.index
GROUP BY activity_day
ORDER BY weekly_active_users desc;

									   
--WEEKLY AND MONTHLY ENGAGEMENT OF USERS WITH SMART DEVICES
SELECT
 extract(month from dailyactivity_merged.date_1) AS activity_month,
 extract( week from dailyactivity_merged.date_1) AS activity_week,		
  COUNT(DISTINCT dailyactivity_merged.id) AS active_users
FROM Public."dailyactivity_merged" INNER JOIN Public."dailycalories_merged"
     ON dailyactivity_merged.index= dailycalories_merged.index
	 INNER JOIN Public."dailyintensities_merged" ON dailycalories_merged.index
	 = dailyintensities_merged.index INNER JOIN Public."dailysteps_merged" ON
	 dailyintensities_merged.index =dailysteps_merged.index
GROUP BY activity_month, activity_week
ORDER BY active_users desc;	

-- Peak usage hours									   
SELECT 
    EXTRACT(HOUR FROM hourlycalories_merged.Time_1) AS Hour_Of_Day,
    SUM(hourlysteps_merged.steptotal) AS Peak_steps,
    SUM(hourlyintensities_merged.totalintensity) AS Peak_intensity, 
    SUM(hourlycalories_merged.calories) AS Peak_calories,
	SUM(heartrate_seconds_merged.value) AS Peak_value_Of_Heart
FROM 
    Public.hourlycalories_merged
INNER JOIN 
    Public.hourlyintensities_merged ON hourlycalories_merged.Index = hourlyintensities_merged.Index
INNER JOIN 
    Public.hourlysteps_merged ON hourlyintensities_merged.Index = hourlysteps_merged.Index
INNER JOIN 
    public.heartrate_seconds_merged ON hourlysteps_merged.Index = heartrate_seconds_merged.Index
GROUP BY 
    Hour_Of_Day
HAVING 
    SUM(hourlysteps_merged.steptotal) > (SELECT AVG(steptotal) FROM Public.hourlysteps_merged) AND
    SUM(hourlyintensities_merged.totalintensity) > (SELECT AVG(totalintensity) FROM Public.hourlyintensities_merged) AND
    SUM(hourlycalories_merged.calories) > (SELECT AVG(calories) FROM Public.hourlycalories_merged) 
ORDER BY Peak_Steps DESC, Peak_intensity DESC, Peak_calories DESC, Peak_value_Of_Heart DESC

--peak usage hour of heart rate feature
SELECT 
      EXTRACT(HOUR FROM heartrate_seconds_merged.Time_1) AS Hour_Of_Day_2,
	  SUM(value) AS Value_heart
FROM public.heartrate_seconds_merged
GROUP BY Hour_Of_Day_2
HAVING SUM(value) > (SELECT AVG(value) FROM Public.heartrate_seconds_merged )
ORDER BY Value_heart

-- USER DAILY ENGAGEMENT OF ACTIVITY, CALORIES, AND STEP TRACKER ( 7 active users)
SELECT  
    *
FROM  
    Public."dailyactivity_merged" 
WHERE totalsteps > ( SELECT AVG(totalsteps)
				     FROM Public."dailyactivity_merged") 
	AND totaldistance > (SELECT AVG(totaldistance)
						 FROM Public."dailyactivity_merged")
	AND veryactiveminutes > (SELECT AVG(veryactiveminutes)
							 FROM Public."dailyactivity_merged")
	AND fairlyactiveminutes > (SELECT AVG(fairlyactiveminutes)
							 FROM Public."dailyactivity_merged")
    AND lightlyactiveminutes > (SELECT AVG(lightlyactiveminutes)
							 FROM Public."dailyactivity_merged")
	AND  sedentaryminutes > (SELECT AVG(sedentaryminutes)
							 FROM Public."dailyactivity_merged")
    AND calories > (SELECT AVG(calories)
							 FROM Public."dailyactivity_merged")


	   
									   
--- ENGAGEMENT WITH SLEEP TRACKER
-- USERS MIN SLEEP RECORDS									   
SELECT *
FROM Public."sleepday_merged"
WHERE (totalsleeprecords,totalminutesasleep,totaltimeinbed) in
	   (SELECT  MIN(totalsleeprecords) AS Min_sleep_record,
	     MIN(totalminutesasleep) AS MIN_sleep_min, MIN(totaltimeinbed) AS
	      Min_time_bed
	     FROM Public."sleepday_merged"
	     GROUP BY date_1)
--MORE THAN AVERAGE SLEEP PATTREN.								   
SELECT *
FROM Public."sleepday_merged"
WHERE  totalsleeprecords > (SELECT AVG(totalsleeprecords)
						    FROM Public."sleepday_merged"
						    ) 
		AND totalminutesasleep > (SELECT AVG(totalminutesasleep)
								  FROM Public."sleepday_merged") 
		AND totaltimeinbed > (SELECT AVG(totaltimeinbed)
							  FROM Public."sleepday_merged")
```

### Insights

- From 12th April up to the 12th May the number of daily active user decreased from 33 to 21. While only in the month of April there was a very slight decrease in Usage of Smart Device. In the month of May the user decrease significantly from 30 to 21.

- In the month of April, each user used the smart device feature for an average of 1,377 minutes. From April to May, there were a total of 840,000 recorded sessions, representing 
  approximately 65% of users. However, in May, usage dropped with only 34% of users (420,000) actively using the smart device. The average usage time per user in May decreased slightly 
  to 1,326 minutes.

- The trend in usage patterns clearly shows that users constantly use the calorie and steps features. This means users consistently count their steps and calories every day, but their 
  average intensity of movement and heart rate is very low. In April, each user's calories burned, heart rate, number of steps taken each day, and intensity of movement were increasing 
  at a very high frequency. But in May, as the number of users dropped significantly, the average calorie burn, heart rate, steps, and intensity also decreased.

- In April, users spent most of their time sitting, averaging between 1,000 and 938 minutes per day. Light movement accounted for an additional 199 to 205 minutes daily. Very active 
  movement was minimal, ranging from only 19 to 28 minutes and varying between users. Despite this, user engagement with the distance feature was high. On average, users covered a light 
  distance between 2.8 and 4 kilometers, with some variation between individuals. Very active distances were lower, averaging between 1 and 1.9 kilometers and also varying between 
  users. Moderate distances fell within a range of 0.3 to 0.7 kilometers.

- In May, we saw a significant decline in engagement with the minute and distance features, as evidenced by the low frequency on the line chart. Sedentary minutes decreased from 953 to 
  632 minutes. The timeframes for lightly active and very active minutes also decreased, dropping from 160 to 98 minutes and 22 to 4 minutes, respectively. There was also a decrease in 
  light activity, with users averaging 3.6 kilometers per day in April and 1.9 kilometers per day in May. Similarly, highly active movement decreased from 1.6 kilometers to 0.3 
  kilometers, and moderate movement fell from 0.7 kilometers to 0.3 kilometers.

- Over the period of two months, most users had an average sleep duration between 369 and 438 minutes.  They might be reading books or using their mobile phones, among other activities. 
  Their average sleep duration varies between 1 and 1.576 minutes, with differences between users. This average gradually decreases as the overall number of users decreases. Around 50% 
  of user didn’t used this feature to track their sleep records from April till may.

- From 8 am to 6 pm, the number of steps increases as men go to work, children go to school, and women do household chores. These are typical working hours. As the number of steps 
  increases, so does the intensity of each step. After 6 pm, both the number of steps and their intensity decrease as people begin to prepare for sleep. The average number of steps 
  ranges from 12,000 to 16,450, and the average intensity ranges from 440 to 602. As steps and intensity increase, calorie burn also increases. Each user experiences their maximum 
  calorie burn during peak hours of steps and intensity. However, heart rate decreases at these peak usage hours. On average, user heart rate is higher when calorie burn, steps, and 
  intensity are lower.

### Recommendations

> Send timely reminder to user to reach their daily activity goal. This would help the user constantlyu track their engagement with the smart device and bella beat would have a more and 
  consistent data which Company can use it for future analysis

> Introduced community events, challenges, and prizes to keep user engaged with the device for example: Bella beat announces that user who has the highest engagement for whole month
  will get a cash prize of 500 dollars. This would motivate users to actively track their health throughout the day and number of user would increase.

> Locate areas in which these smart devices are mostly used and target people of that area through marketing campaigns. implement attractive marketing srategies that would encourge user
   to buy Bella beat smart device product. 

> Introducing a personalized AI Assistant that tracks and updates user on daily calories, steps, and provides tailored recommendations to improve sleep quality. Additionally, it 
  suggests personalized workout sessions and sends reminders to ensure a consistent sleep schedule, such as going to bed by 10 pm and waking up at 6 am.

> Take feedback from user who stopped using smart device through in App notifcations, emails, or surveys and ask them what are the reasons that let them discontinue bella
  beat smart device an based on the feedback work on those areas or features of smart device.































































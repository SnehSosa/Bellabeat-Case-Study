In this project I hav worked on Fitbit Database for a Fitness Company-Bellabeat. I have used SQL and EXCEL for data cleaning and visualization.




--Calculating 
--Number of users tracking physical activities and daily averages
SELECT 
   COUNT(DISTINCT Id) AS users_tracking_activity,
   AVG(CAST(TotalSteps AS decimal)) AS average_steps,
   AVG(CAST(TotalDistance AS decimal)) AS average_distance,
   AVG(CAST(Calories AS decimal)) AS average_calories
FROM
    bellabeat.dbo.daily_activity

--Number of users tracking heart rate
SELECT 
   COUNT(DISTINCT Id) AS users_tracking_heartrate,
   AVG(Value) AS average_heartrate,
   MIN(Value) AS minimum_heartrate,
   MAX(Value) AS maximum_heartrate
FROM
    bellabeat.dbo.heartrate_seconds

--Number of users tracking weight
SELECT 
   COUNT(DISTINCT Id) AS users_tracking_weight,
   AVG (CAST(WeightKg AS decimal)) AS average_weight,
   MIN(CAST(WeightKg AS decimal)) AS minimum_weight,
   MAX(CAST(WeightKg AS decimal)) AS maximum_weight
FROM
    bellabeat.dbo.weightLogInfo

--Number of users tracking sleep
SELECT 
   COUNT(DISTINCT Id) AS users_tracking_sleep,
   AVG(TotalMinutesAsleep)/60.0 AS average_hours_sleep,
   MIN(TotalMinutesAsleep)/60.0 AS minimum_hours_sleep,
   MAX(TotalMinutesAsleep)/60.0 AS maximum_hours_sleep,
   AVG(TotalTimeInBed)/60 AS average_hours_inbed
FROM
    bellabeat.dbo.sleep_day

*	Calculating to check if there is any correlation between the activities and days
SELECT 
ActivityDate,
AVG (Bellabeat.dbo.daily_activity.TotalSteps) AS Steps_Average,
AVG (Bellabeat.dbo.daily_activity.SedentaryMinutes) AS Sedentary_Average,
AVG(Bellabeat.dbo.daily_activity.LightlyActiveMinutes) AS Lightly_Average,
AVG (Bellabeat.dbo.daily_activity.FairlyActiveMinutes) AS Fairly_Average,
AVG(Bellabeat.dbo.daily_activity.VeryActiveMinutes) AS VeryActive_Average

FROM
  Bellabeat.dbo.daily_activity

WHERE 
 Bellabeat.dbo.daily_activity.TotalSteps <> 0

GROUP BY ActivityDate

ORDER BY Steps_Average;


Cleaning daily activity and calculating average 
Drop table
IF EXISTS(SELECT * 
           FROM bellabeat.dbo.daily_activity_cleaned)
DROP TABLE bellabeat.dbo.daily_activity_cleaned

--Create table to convert datatypes
CREATE TABLE bellabeat.dbo.daily_activity_cleaned
(Id FLOAT, ActivityDate DATETIME2(7), TotalSteps INT, TotalDistance FLOAT, LoggedActiveDistance FLOAT,VeryActiveDistance FLOAT, ModeratelyActiveDistance FLOAT, 
	LightlyActiveDistance FLOAT, SedentaryActiveDistance FLOAT, VeryActiveMinutes INT, FairlyActiveMinutes INT, LightlyActiveMinutes INT, SedentaryActiveMinutes INT,
	Calories FLOAT)

INSERT INTO bellabeat.dbo.daily_activity_cleaned
	(Id, ActivityDate, TotalSteps, TotalDistance, LoggedActiveDistance,VeryActiveDistance, ModeratelyActiveDistance, 
	LightlyActiveDistance, SedentaryActiveDistance, VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes, SedentaryActiveMinutes,
	Calories)

SELECT 
	Id,
	ActivityDate,
	CAST(TotalSteps AS FLOAT) AS TotalSteps,
	CAST(TotalDistance AS FLOAT) AS TotalDistance,
	CAST(LoggedActivitiesDistance AS FLOAT) AS LoggedActiveDistance,
	CAST(VeryActiveDistance AS FLOAT) AS VeryActiveDistance,
	CAST(ModeratelyActiveDistance AS FLOAT) AS ModeratelyActiveDistance,
	CAST(LightActiveDistance AS FLOAT) AS LightActiveDistance,
	CAST(SedentaryActiveDistance AS FLOAT) AS SedentaryActiveDistance,
	CAST(VeryActiveMinutes AS FLOAT) AS VeryActiveMinutes,
	CAST(FairlyActiveMinutes AS FLOAT) AS FairlyActiveMinutes,
	CAST(LightlyActiveMinutes AS FLOAT) AS LightlyActiveMinutes,
	CAST(SedentaryMinutes AS FLOAT) AS SedentaryActiveMinutes,
	CAST(Calories AS FLOAT) AS Calories

FROM bellabeat.dbo.daily_activity



--Determine when users were mostly active
SELECT*
FROM bellabeat.dbo.hourly_activity


SELECT 
 DISTINCT (CAST(ActivityHour AS TIME)) AS activity_time,
 AVG (TotalIntensity) OVER (Partition By DATEPART (HOUR,ActivityHour)) AS average_intensity,
 AVG(METs/10.0) OVER (Partition By DATEPART (HOUR, ActivityHour)) AS average_METs
 FROM
  bellabeat.dbo.hourly_activity AS hourly_activity
 
JOIN bellabeat.dbo.minuteMETsNarrow AS METs
ON
  hourly_activity.Id = METs.Id AND
  hourly_activity.ActivityHour = Mets.ActivityMinute
ORDER BY 
average_intensity DESC

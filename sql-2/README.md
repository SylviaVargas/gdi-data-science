# SQL II course notes

# Created by: Faiza Khan

## Course description:

Topics in this course will include: advanced joins and subqueries, window functions, and common table expressions (CTEs). In this course, an online SQL database tool will be used to teach students how to write queries

Tool used to teach the course: [DB fiddle](https://www.db-fiddle.com/)

# Session 1:

### Topics:

- Types of joins
- Joins on multiple keys
- CASE statement

- Joins review (slide 58 in this presentation)
- Write queries using the set of tables provided in [student data](https://www.db-fiddle.com/f/6eXpPSRFQgzdKUCjSsbSF4/9)

# Sessions 2 & 3:

### Topic:

- Window functions: ROW_NUMEBR(), RANK(), DENSE_RANK()

DB Fiddle link to bike data exercise: https://www.db-fiddle.com/f/eZ5mSe2SZBqFfqLfSf5YzJ/3

### Questions: 

+ 1- What are the top 3 earliest times on which someone took a ride
Select *
From bike_trips
Order
   by start_time
 Limit 3;

+ 2- Arrange rides by start_date and start_time

Select *
  From bike_trips
 Order
    by start_date, start_time

+ 3- Assign ranks to the rides based on start_date and start_time
 
Select *,
       ROW_NUMBER() OVER (Order by start_date, start_time) as ride_rank
  From bike_trips

+ 4- Assign ranks to rides by each rider based on their start date

Select *,
      ROW_NUMBER() OVER (Partition by rider_id Order by start_date) as ride_rank
  From bike_trips;

+ 5- For each rider, find the earliest date a ride was taken

Select *
  From (
Select rider_id,
       start_date, 
       ROW_NUMBER() OVER (Partition by rider_id Order by start_date) as ride_rank
  From bike_trips) A
  where A.ride_rank = 1;

+ 6- For each rider, find the latest date a ride was taken
Select *
  From (
Select rider_id,
       start_date, 
       ROW_NUMBER() OVER (Partition by rider_id Order by start_date DESC) as ride_rank
  From bike_trips) A
  where A.ride_rank = 1;

+ 7-  Compute the duration of each ride
Select rider_id,
       start_date,
       CAST(end_time - start_time AS time) as duration
  From bike_trips

+ 8-  Order the rides by start_date and duration
Select A.*,
       ROW_NUMBER() OVER (Order by A.start_date, A.duration) as ride_rank
 From (
Select rider_id,
       start_date,
       CAST(end_time - start_time AS time) as duration 
  From bike_trips) A

+ 9- Order the durations within each rider
Select A.*,
      ROW_NUMBER() OVER (Partition by rider_id Order by A.duration) as ride_rank
 From (
Select rider_id,
       start_date,
       CAST(end_time - start_time AS time) as duration 
  From bike_trips) A;

+ 10- For each rider, select the ride with the highest duration
Select B.*
  From (
Select A.*,
       ROW_NUMBER() OVER (Partition by rider_id Order by A.duration DESC) as ride_rank
 From (
Select rider_id,
       start_date,
       CAST(end_time - start_time AS time) as duration 
  From bike_trips) A) B
where B.ride_rank = 1;

+ 11- Select *,
       RANK() OVER (Order by start_date)
  From bike_trips;

Rank skips value after more than one row has been assigned the same rank

+ 12- Select *,
       DENSE_RANK() OVER (Order by start_date)
  From bike_trips;

# Session 4:

### Topic:
- CTEs (Common Table Expressions)

DB Fiddle link to student data: https://www.db-fiddle.com/f/6eXpPSRFQgzdKUCjSsbSF4/9

### Question: Compute number of applications submitted by students

Subquery:
Select A.sID,
       COUNT(A.college_name) as num_apps
  From (
Select St.student_id as sID,
       St.student_name as sName,
       college_name,
       major,
       decision
  From Student St
  Left
  Join Applications App
    On St.student_id = App.student_id) A
 Group
    by 1
 Order
    by num_apps

Turned into CTE:

With result as 
   (
Select St.student_id as sID,
       St.student_name as sName,
       college_name,
       major,
       decision
  From Student St
  Left
  Join Applications App
    On St.student_id = App.student_id) 
    
Select sID,
       COUNT(college_name) as num_apps
  From result
 Group 
    by 1
 Order
    by num_apps;

### Question: Compute average number of applications & average number of distinct majors a student applied to

With join_result as 
   (
Select St.student_id as sID,
       St.student_name as sName,
       college_name,
       major,
       decision
  From Student St
  Left
  Join Applications App
    On St.student_id = App.student_id),
count_result as
	(
      	  Select sID,
            		COUNT(DISTINCT major) as num_major,
       		COUNT(college_name) as num_apps
  		From join_result
 	   Group 
    	  by 1
	   Order
    	  by num_apps)
          
Select AVG(num_major), 
           AVG(num_apps)
  From count_result;

### Question: Compute average number of applications & average number of distinct majors a student applied to & average number of accepted applications

With join_result as 
   (
Select St.student_id as sID,
       St.student_name as sName,
       college_name,
       major,
       decision
  From Student St
  Left
  Join Applications App
    On St.student_id = App.student_id),
count_result as
	(
      Select sID,
            COUNT(DISTINCT major) as num_major,
            COUNT(college_name) as num_apps,
            SUM(CASE WHEN decision = 'Y' THEN 1 ELSE 0 END) as num_accepts
  		From join_result
 	   Group 
    	  by 1
	   Order
    	  by num_apps)
          
Select AVG(num_major),
       AVG(num_apps),
       AVG(num_accepts)
  From count_result;

Following questions are based on [bike data](https://www.db-fiddle.com/f/eZ5mSe2SZBqFfqLfSf5YzJ/3)

### Question: for each rider, finding ride with least/most duration

With duration_result as
  (Select *,
          CAST(end_time - start_time AS time) as duration
     From bike_trips),
   rownumber_result as 
   (Select *,
           ROW_NUMBER() OVER (Partition by rider_id Order by duration ASC/DESC) as rn
      From duration_result)
 
Select *
  From rownumber_result
 where rn = 1 ;

### Question: For each rider, find the earliest date a ride was taken

With rownumber_result as 
   (Select *,
           ROW_NUMBER() OVER (Partition by rider_id Order by start_date) as rn
      From bike_trips)
 
Select *
  From rownumber_result
 where rn = 1;

### Question: for each rider, find the earliest date and the earliest time a ride was taken

With rownumber_result as 
   (Select *,
           ROW_NUMBER() OVER (Partition by rider_id Order by start_date) as rn,
           ROW_NUMBER() OVER (Partition by rider_id Order by start_time) as rn_time
      From bike_trips)
 
Select *
  From rownumber_result
 where rn = 1 OR rn_time = 1 ;


## Used Cars in United Kingdom Analysis

### Table of Contents
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Tool](#tool)
- [Data Cleaning process](#data-cleaning-process)
- [Data Cleaning](#data-cleaning)
- [Data Analysis](#data-analysis)



### Project Overview
This data analysis project aims to provide insights about used cars in United Kingdom. By cleaning and analyzing various aspects of this dataset, i seek to identify trends, make data driven recommendations and gain a deeper knowledge.

### Data Source
Used Cars in UK: The primary dataset used for this analysis is the 'used_uk_cars.csv' file, containing detailed information.

### Tool
- MYSQL
   - Data Cleaning
    - Data Analysis

### Data Cleaning process
In this phase, i performed the following tasks;
- Data Importation
- Data Cleaning

### Data Cleaning 
``` MYSQL
-- Data Cleaning
-- To check for duplicates
SELECT title, registration_year, price, engine,seats, count(*) 
	FROM used_cars_uk GROUP BY title, registration_year, price, engine, seats HAVING count(*) > 1;
    
    -- to rename column to use it as unique identifier
    ALTER TABLE used_cars_uk
    change myunknowncolumn s_n int;
    
    
-- Removing duplicates
DELETE t1
FROM used_cars_uk as t1
JOIN (SELECT  title, registration_year, price, engine,seats, min(s_n) as min_serial_number
from used_cars_uk
group by  title, registration_year, price, engine,seats
having count(*) > 1)
as t2 on t1.title = t2. title
	and t1.registration_year = t2.registration_year
    and t1.price = t2.price
    and t1.engine = t2.engine
    and t1.seats = t2.seats
    and t1.s_n > t2.min_serial_number;

-- Check for nulls
SELECT  engine, `previous owners`, `emission class`, `service history`
    from used_cars_uk
    where title = '' or
    registration_year = ''or
    seats  = '' or
    `previous owners`  = '' or
    `fuel type`  = '' or
	`body type`  = '' or
    gearbox  = '' or
    doors  = '' or
    `emission class`  = '' or
    `service history`= '';
    
    
    alter table USED_CARS_UK
    change `Service history` service_history text;
    
    alter table USED_CARS_UK
    change `BODY TYPE` Body_type text;
    
    alter table USED_CARS_UK
    change `fuel type` Fuel_type text;
    
    alter table USED_CARS_UK
    change `mileage(miles)` Mileage_miles int;
    
    alter table USED_CARS_UK
    change `PREVIOUS OWNERS` Previous_owners text;
    
    alter table USED_CARS_UK
    change `emission class` Emission_class text;
    
    -- to replace blanks in service history column
    update  USED_CARS_UK
    set Service_history = case when Service_history <> 'full' then 'no repair'
    end;
    
    update used_cars_uk 
    set service_history = case when service_history is null then 'full'
    else 'no repair'
    end;
    
    -- to replace blank rows in previous_owner column with null
    update used_cars_uk
    set previous_owners = null
    where previous_owners = '';
    
    -- to replace the null values in previous_owners with 0
    update used_cars_uk
    set previous_owners = coalesce(previous_owners, 0 );
    
    -- to change previous_owners datatype
    alter table used_cars_uk
    modify column previous_owners int;
    
    -- to replace blank rows with 0liters
    update used_cars_uk
    set engine = '0L'
    where engine ='';
    
    -- to change datatype for registration_year column from int to year
    alter table used_cars_uk
    modify column registration_year year;
    
    -- to remove L(liters) from engine column and change the data type
    update used_cars_uk 
   SET engine = replace(engine, 'L', '')
   where engine like '%l%';
   
   alter table used_cars_uk
   modify column engine decimal(2,1);
   
    
    select * from used_cars_uk ;
    
   -- to check if there are still blank rows in the table
   select * from used_cars_uk
   where coalesce(s_n ,title , Price , 
Mileage_miles ,registration_year , previous_owners,  Fuel_type,  Body_type , engine,  Gearbox , 
Doors,  Seats,  Emission_class ,service_history) = '';

-- to remove blanks that are maybe whitespaced or special characters from that column
select * from used_cars_uk
where trim(ifnull(emission_class, '')) = '';

-- to replace them with 0
update used_cars_uk
set emission_class = 0
where trim(ifnull(emission_class, '')) = '';
```


### Data Analysis
``` MYSQL
# ANALYSIS
-- 1) total number of cars 
SELECT distinct count(*) from used_cars_uk;

-- 2) Top 5 and bottom 5 Car models with the highest price
select title, price
from used_cars_uk group by title, price
order by price desc limit 5;

select title, price
from used_cars_uk group by title, price
order by price asc limit 5;

# Does the mileage of a car determine the price? the higher the mileage the higher the price?
-- 3) car model, price and mileage
select title, price, mileage_miles
from used_cars_uk order by  mileage_miles desc;

# Does the amount of previous ownerws determine the price? are second hand vehicles lesser than brand new?
-- 4) car model, previous owners, price
select title, price, previous_owners
from used_cars_uk order by price desc;

#
-- 5) Car model, engine, mileage_miles

# is emission class dependent on what type of engine and fuel type
-- 6) car model, fuel type, engine, emission_class

select title, fuel_type, engine, emission_class
from used_cars_uk  order by emission_class desc  ;
# petrol and disel fueled cars have emission_class of 0-6, while electric have 0 emission_class

# Does the type of engine determine the price
-- 7)engines, prices
select 
	case when engine >= 0 and engine <= 2 then '0-2'
    when engine > 2 and engine <= 3 then '2-3'
    when engine > 3 and engine <= 4 then '3-4'
    when engine > 4 and engine <= 5 then '4-5'
    when engine > 5 and engine <= 6 then '5-6'
    when engine > 6 and engine <= 7 then '6-7'
    end as engine_category,
  avg(price) as Average_price
from used_cars_uk group by engine_category order by Average_price desc;

# Does a car having multiple previous owners mean they have been service_history
-- 8) car model, service history, previous owners
select title, service_history, previous_owners
from used_cars_uk order by previous_owners;
# irrespective of the number of previous owners some still have full service history and some don't repair at all

select title, service_history, count(*)
from used_cars_uk where service_history = 'no repair' group by title, service_history order by count(*);

select title, service_history, count(*)
from used_cars_uk where service_history = 'full' group by title, service_history order by count(*);

-- 9) total number of cars that have and have not been serviced
select service_history, count(*)
from used_cars_uk where service_history = 'no repair' group by  service_history order by count(*);

select  service_history, count(*)
from used_cars_uk where service_history = 'full' group by  service_history order by count(*);
# majority of the cars have service history


-- what is the average of those that have and does not have service history
select  avg(price),service_history, count(*)
from used_cars_uk where service_history = 'full' group by  service_history order by count(*);

select avg(price),service_history, count(*)
from used_cars_uk where service_history = 'no repair' group by  service_history order by count(*);

# those with service history have higher average price

-- sum of all the serviced and non serviced cars
select  sum(price),service_history, count(*)
from used_cars_uk where service_history = 'full' group by  service_history order by count(*);

select sum(price),service_history, count(*)
from used_cars_uk where service_history = 'no repair' group by  service_history order by count(*);

## sum of price of cars without service history are higher than those with service history

-- total count of cars available by their registration year
select registration_year, count(*)
from used_cars_uk group by registration_year order by count(*) desc;
## 2011 has the highest number of cars, 1988-1993 have the least number of cars
```

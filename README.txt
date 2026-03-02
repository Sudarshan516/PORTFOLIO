create database project;
use project;
show tables;

select  * from fact_bookings;
SET GLOBAL local_infile = 1;
SHOW GLOBAL VARIABLES LIKE 'local_infile';



LOAD DATA LOCAL INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/fact_bookings.csv'
INTO TABLE fact_bookings
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;


-- total Revenue
select concat(round(sum(revenue_realized)/1000000,2),'M') as `Total Revenue` from fact_bookings;

-- total booking
select concat(round((sum(successful_bookings)/1000),2),'k')`Total Bookings` from fact_aggregated_bookings;

-- Average rating
select avg(ratings_given) as `Average Rating`from fact_bookings;

-- Occupancy
select concat(round((sum(successful_bookings)/sum(capacity))*100,2),'%') as Occupancy from fact_aggregated_bookings;

-- cancellation%
SELECT 
   concat( round(COUNT(CASE WHEN booking_status = 'Cancelled' THEN 1 END) * 100
    / COUNT(*),2),'%') AS `Cancellation`
FROM fact_bookings;

-- city wise revenue
select dim_hotels.city,concat(round(sum(fact_bookings.revenue_realized)/1000000,2),'M') as `Total revenue` from dim_hotels
join
 fact_bookings on dim_hotels.property_id = fact_bookings.property_id
 group by dim_hotels.city
 order by `Total Revenue` desc;
 select * from dim_date;
 
-- Day Type Revenue
 SELECT dim_date.day_type,
      concat(round( SUM(fact_bookings.revenue_realized)/1000000,2),'M') AS `Total Revenue` from dim_date
JOIN fact_bookings
  ON dim_date.date = STR_TO_DATE(fact_bookings.date, '%Y/%m/%d')
GROUP BY dim_date.day_type
order by `Total Revenue`;

-- class wise revenue
select dim_rooms.room_class,concat(round(sum(fact_bookings.revenue_realized)/1000000,2),'M') as `Total Revenue`  from dim_rooms
join 
	fact_bookings on dim_rooms.room_id = fact_bookings.room_id
    group by dim_rooms.room_class
    order by `Total Revenue` desc;
    
-- hotel wise revenue
select dim_hotels.property_name,concat(round(sum(fact_bookings.revenue_realized)/1000000,2),'M') as `Total revenue` from dim_hotels
join
 fact_bookings on dim_hotels.property_id = fact_bookings.property_id
 group by dim_hotels.property_name
 order by `Total Revenue` ;

-- booking status analysis
select  booking_status,concat(round(sum(revenue_realized)/1000000,2),'M') as `Total Revenue`  from fact_bookings
group by booking_status;

-- weekly trend
select dt.`week no`,concat(round(sum(revenue_realized)/1000000,2),'M')as `Total Revenue`,
    concat( round(COUNT(CASE WHEN booking_status = 'Cancelled' THEN 1 END) * 100
    / COUNT(*),2),'%') AS `Cancellation`
    from dim_date dt
left  join 
 fact_bookings fb on dt.date =  STR_TO_DATE(fb.date, '%Y/%m/%d')
   left join
   fact_aggregated_bookings fab on dt.date = STR_TO_DATE(fab.date, '%d-/%b-%y')
 group by dt.`week no`
 order by  dt.`week no`;


 

Case Study with SQL & Tableau – Cyclistic Bike Share
====================================================

<div style="text-align:center">
    <img src="https://ningjoanne.files.wordpress.com/2024/05/screenshot-2024-05-02-104119.png?w=197" alt="Your Image Description">
</div>

* * *

Business Prompt
---------------

Cyclistic is a fictional bike-share company based in Chicago. The director of marketing believes that the company’s future success hinges on maximizing the number of annual memberships. Therefore, the objective is to gain insight into how casual riders and annual members use Cyclistic bikes differently and convert casual riders into annual members.

* * *

Step 1. Ask Question
--------------------

**Key Task:** Based on the provided [historical trip data](https://divvy-tripdata.s3.amazonaws.com/index.html) to gain the insight and understand the difference between casual and member riders.

**Guiding Question:**

1.  How do annual members and casual riders use Cyclistic bikes differently?
2.  Why would casual riders buy Cyclistic annual memberships?
3.  How can Cyclistic use digital media to influence casual riders to become members?

* * *

Step 2. Prepare Data
--------------------

##### **Evaluate the validity of the Data**

This case study utilizes historical trip data, which encompasses user information such as user ID and membership type, along with the details of each journey. The study specifically analyses trip data recorded from March 2023 to February 2024.

*   **Reliable:** The 12 months data contains 5,707,168 unique user ID which is over the minimum requirement of the sample size.
*   **Original:** Cyclistic is a fictional company, so the dataset is provided by Motivate International Inc.
*   **Comprehensive:** The data has over a million rows with user ID, and all the detail of their riding journey.
*   **Current:** The data spans from March 2023 to February 2024.
*   **Cited:** Datasets is made by Motivate International Inc.

* * *

Step 3. Process Data
--------------------

Steps and scenarios show below:

1\. Create a new table and merge data from the past 12 months. The result shows that we have over 5 million rows.

    CREATE TABLE tripdata_12m AS
    
    SELECT *
    FROM tripdata202303
    UNION ALL
    SELECT *
    FROM tripdata202304
    UNION ALL
    SELECT *
    FROM tripdata202305
    UNION ALL
    SELECT *
    FROM tripdata202306
    UNION ALL
    SELECT *
    FROM tripdata202307
    UNION ALL
    SELECT *
    FROM tripdata202308
    UNION ALL
    SELECT *
    FROM tripdata202309
    UNION ALL
    SELECT *
    FROM tripdata202310
    UNION ALL
    SELECT *
    FROM tripdata202311
    UNION ALL
    SELECT *
    FROM tripdata202312
    UNION ALL
    SELECT *
    FROM tripdata202401
    UNION ALL
    SELECT *
    FROM tripdata202402
    

![](https://ningjoanne.files.wordpress.com/2024/04/total_12m.png?w=1024)

2\. Check the duration of time users ride the bike. Unreasonable results are apparent in both the maximum and minimum durations.

    SELECT  
    TO_CHAR(AVG(ended_at - started_at), 'HH24:MI:SS') AS avg_ride_time,
    MAX(ended_at - started_at) AS max_ride_time,
    MIN(ended_at - started_at) AS min_ride_time
    FROM tripdata_12m

![](https://ningjoanne.files.wordpress.com/2024/04/duration.png?w=395)

3\. Clean data and Create a table for future use.

*   Add the duration column and remove the duration < 0 second and > 24 hr.
*   Extract month, day of week and time data to separate columns.
*   Remove null column in start\_station\_name, star\_station\_id, end\_station\_name, end\_station\_id.
*   Remove longitude and latitude as we won’t need these columns in the following analysis.
*   Rename member\_casuaul column to membershiptype.

    CREATE TABLE tripdata_12m_new AS
    
    SELECT ride_id,
    		rideable_type,
    		started_at,
    		TO_CHAR(started_at, 'Month') AS started_month,
    		TO_CHAR(started_at, 'Day') AS started_dow,
    		CAST(started_at AS TIME) AS started_time,
    		ended_at,
    		TO_CHAR(ended_at, 'Month') AS ended_month,
    		TO_CHAR(ended_at, 'Day') AS ended_dow,
    		CAST(ended_at AS TIME) AS ended_time,
    		(ended_at - started_at) AS duration,
    		start_station_name,
    		start_station_id,
    		end_station_name,
    		end_station_id,
    		member_casual AS membership_type		
    FROM tripdata_12m
    WHERE start_station_name IS NOT NULL 
    		AND start_station_id IS NOT NULL
    		AND end_station_name IS NOT NULL 
    		AND end_station_id IS NOT NULL
                    AND (ended_at - started_at) > INTERVAL '0' SECOND
    		AND (ended_at - started_at) < INTERVAL '24' HOUR;
    

4\. The result shows 4,331,781 rows in the new table after cleaning and the data is ready to export and using in Tableau.

![](https://ningjoanne.files.wordpress.com/2024/04/new_12m.png?w=1024)

* * *

Step 4. Analyse Data
--------------------

##### **General Idea**

📊**Click [here](https://public.tableau.com/shared/WRTXC7SF4?:display_count=n&:origin=viz_share_link) to check the dashboard on Tableau**.

Firstly, let’s have some general idea from the datasets. The membership between casual and member is 35% : 65% and there are three different types of bikes, which classic bike is used the most and then electric bike.

![](https://ningjoanne.files.wordpress.com/2024/04/dashboard-4.png?w=999)

##### **Assumption**

Before diving straight into the data, I would like to make some hypotheses and use data to verify them.

1.  Do casual and membership riders use the bike differently during different months and days of the week?
2.  What are the most popular start stations and routes among casual and membership riders?
3.  What’s the preference for bike type between casual and membership riders?
4.  Which customer type has a longer duration?

##### **Verify the assumptions**

❓Assumption 1: Do casual and membership riders use the bike differently during different months and days of the week?

📈Answer: The data indicates that both casual and membership riders tend to use bikes more frequently in summer and less in winter. However, casual riders show a preference for weekend rides, particularly on Saturdays, while membership riders use bikes more on weekdays.

![](https://ningjoanne.files.wordpress.com/2024/04/user-by-month-1.png?w=1004)

![](https://ningjoanne.files.wordpress.com/2024/04/user-by-dow-1-1.png?w=707)

On weekdays, there are always more membership riders than casual riders, but on weekends, the numbers of casual and membership riders are nearly equal.

![](https://ningjoanne.files.wordpress.com/2024/04/user-by-dow_1-1-2.png?w=1024)

* * *

❓Assumption 2: What are the most popular start stations and routes among casual and membership riders?

📈Answer: According to the data below, the preference for start stations varies between casual and membership riders. The most popular station for casual riders is Streeter Dr & Grand Ave, while for membership riders, it is Clinton St & Washington Blvd.

![](https://ningjoanne.files.wordpress.com/2024/04/popular_station.png?w=999)

Additionally, casual riders tend to rent and return the bike at the same station more frequently, whereas membership riders do not.

![](https://ningjoanne.files.wordpress.com/2024/04/popular_route.png?w=999)

❓Assumption 3: What’s the preference for bike type between casual and membership riders?

📈Answer: Both casual and membership members prefer to ride classic bike than electric type.

![](https://ningjoanne.files.wordpress.com/2024/04/story-1-3.png?w=1015)

❓Assumption 4: Which customer type has a longer duration?

📈Answer: Casual riders tend to ride longer than membership riders.

    SELECT 
        membership_type,
        TO_CHAR(AVG(duration), 'HH24:MI:SS') AS avg_ride_time
    FROM 
        tripdata_12m_new
    GROUP BY 
        membership_type;
    	

![](https://ningjoanne.files.wordpress.com/2024/04/membership_averageride.png?w=281)

* * *

Step 5. Sharing insight and Act
-------------------------------

Based on the above observations, the difference between casual and membership riders are obvious and the recommendations are listed as below:

*   Insight 1: casual riders prefer to use bike during summer time and weekend.
    *   Recommendation: Cyclistic’s marketing team can focus on targeting casual riders with weekend campaigns during the summer.
*   Insight 2: Casual riders favour Streeter Dr & Grand Ave station, often opting to rent and return bikes at the same spot, typically located in travel spots like shores, parks, and harbours.
    *   Recommendation: Cyclistic’s marketing team should set up more point of sales at these stations, offering promotions like one-month membership trials, to entice casual riders to become members.
*   Insight 3: Both casual and membership members prefer to ride classic bike than electric type.
    *   Recommendation: Keep investing and maintaining the classic bikes.
*   Insight 4: Casual riders ride for longer durations compared to membership riders.
    *   Recommendation: Review the pricing structure and inform casual riders that becoming a member is more cost-effective if they ride for extended periods.

* * *

#### **Limitation**

1.  The [historical trip data](https://divvy-tripdata.s3.amazonaws.com/index.html) doesn’t include users’ gender and age group, which could bias the marketing plan implementation if any.
2.  Blank rows in different columns might affect the results of this case study

* * *

#### **Reference**

*   [https://divvy-tripdata.s3.amazonaws.com/index.html](https://divvy-tripdata.s3.amazonaws.com/index.html)
*   [https://www.coursera.org/professional-certificates/google-advanced-data-analytics?utm\_medium=sem&utm\_source=gg&utm\_campaign=b2c\_emea\_google-advanced-data-analytics\_google\_ftcof\_professional-certificates\_arte\_feb\_24\_dr\_geo-multi\_sem\_rsa\_gads\_lg-all&campaignid=21008476799&adgroupid=158313472989&device=c&keyword=google%20data%20analytics&matchtype=p&network=g&devicemodel=&adposition=&creativeid=690404375441&hide\_mobile\_promo&gad\_source=1&gclid=CjwKCAjwt-OwBhBnEiwAgwzrUvkHQFwwuFK\_v1\_c6d9emERfTLwvNJ4itbCQNca9A9XHDNyma\_OvVBoCLocQAvD\_BwE](https://www.coursera.org/professional-certificates/google-advanced-data-analytics?utm_medium=sem&utm_source=gg&utm_campaign=b2c_emea_google-advanced-data-analytics_google_ftcof_professional-certificates_arte_feb_24_dr_geo-multi_sem_rsa_gads_lg-all&campaignid=21008476799&adgroupid=158313472989&device=c&keyword=google%20data%20analytics&matchtype=p&network=g&devicemodel=&adposition=&creativeid=690404375441&hide_mobile_promo&gad_source=1&gclid=CjwKCAjwt-OwBhBnEiwAgwzrUvkHQFwwuFK_v1_c6d9emERfTLwvNJ4itbCQNca9A9XHDNyma_OvVBoCLocQAvD_BwE)


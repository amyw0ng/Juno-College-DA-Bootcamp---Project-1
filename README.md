# Juno College DA Bootcamp - Project 1


## Background

Let's get some background! We've been hired by a mobile gaming company and on the one-year anniversary, we've been tasked with investigating the player retention to learn how our mobile game has performed this past year.

For our investigation we were provided with:
- Match information, including the players who matched against each other, and the outcome
- Player information, including information like the player's age and when they joined


## 30-Day Rolling Retention Query

First off, we defined retention based on whether or not a player played a game 30 days after joining the game. In order to determine which players fall into this category, we used:
```
WITH player_info_retention_stat AS (
    SELECT 
        DISTINCT p.player_id,
        p.joined,
        IF(MAX(day) OVER (PARTITION BY p.player_id) >= joined+30, 1, 0) AS retention_status,
    FROM `juno-da-bootcamp-project-1.raw_data.player_info` AS p
    LEFT JOIN `juno-da-bootcamp-project-1.raw_data.matches_info` AS m
    ON p.player_id = m.player_id)
```
The above allowed us to create a temporary table called **player_info_retention_stat**, using _WITH - AS_, that returns a **1** if the player was retained (their latest game played was after 30 days of joining the game) or a **0** if they were not retained (they did not play a game after 30 days of joining). 

We then used:
```
SELECT 
    *,
    CASE 
        WHEN LAG(fractional_retention) OVER (ORDER BY join_day) > 0 THEN
        (fractional_retention - LAG(fractional_retention) OVER (ORDER BY join_day)) / LAG(fractional_retention) OVER (ORDER BY join_day)
        WHEN join_day = 1 THEN 0
        ELSE 0
        END AS growth_rate
FROM (    
    SELECT 
        joined AS join_day,
        COUNT(joined) AS total_joined,
        SUM(retention_status) AS total_retained,
        ROUND(SUM(retention_status)/COUNT(joined),4) AS fractional_retention
    FROM player_info_retention_stat AS pr
    GROUP BY joined
    ORDER BY joined)
ORDER BY join_day
```
In the query above, we grouped the player data by the day on which they joined the game to calculate:
- **total_joined**: The total number of players who joined each day 
- **total_retained**: The total number of players who were retained each day based on the 30-day retention
- **fractional_retention**: The fraction of players that were retained given the total number of players each day

This **fractional_retention** was then used with the _CASE - WHEN_ to calculate the **growth_rate** per day for our final table. This was calculated based on:
```
(fractional_retention current day MINUS fractional_retention of previous day) DIVIDED BY fractional_retention of previous day
```
All of the above allowed us to explore the 30-day rolling retention for the game over the past year.


### 30-Day Rolling Retention Analysis

<p align="center"><img width="" height="" src="https://github.com/amyw0ng/Juno-College-DA-Bootcamp---Project-1/blob/main/Percent%20Retention%20and%20Growth%20Rate%20Graph.png?raw=true"></p>

Given we were working with a 30-day rolling retention, we excluded the last 30 days of the year from our analysis because those who joined in the last 30 days of the year would have automatically been deemed not-retained given we were only working with data up to the end of the year.

From our data above, we saw an average 30-day retention of 65.62% per day for our mobile game over the past year. This was also quite consistent for the whole year as per our data above. This is great considering the industry bench mark for 30-day retention (according to https://www.geckoboard.com/best-practice/kpi-examples/retention-rate/) sits at 42%. Our mobile app has been performing well for it's first year!

In looking at the growth rate, while our retention has been great, we do see that there isn't much change in growth over the course of the year and it has been quite stagnant. The changes we may have implemented this year to the game doesn't seem to have had an impact on the player retention so far so it would be good to explore other incentive systems to make further improvements. This may also be a sign for us to start directing our energy into long-term retention for those who have stayed with us past the 30-day retention benchmark.


## Effects of Winning Query

For our second investigation, we wanted to exploring whether wins played any part in incentivizing players to keep playing. Particularly, whether win-streaks were the same or 



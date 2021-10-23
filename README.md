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



+++
title = "Introducing Simple Shot Quality Model (SSQM) v2.0"
date = "2024-02-18"
description = "My second attempt at creating a shot quality model using publicly available NBA data"

draft = false

[taxonomies]
tags = ["Shooting","Shot Quality","SSQM"]
categories = ["NBA"]
[extra]
math = true
math_auto_render = true
keywords = "Shooting, Shot Quality"
+++

My second attempt at creating a shot quality model using publicly available NBA data. First attempt is [here](https://blog.sradjoker.cc/posts/nba-shot-quality-1/).

## Introduction
As I noted in my previous [blog](https://blog.sradjoker.cc/posts/nba-shot-quality-1/), only shot location is not a holistic way of looking at shot quality and there should be other factors like shot type, defender distance, touch time, dribble etc. which need to be incorporated in the model. So this model, which I named `SSQM v2.0` is an attempt to include those factors. 

## Methodology
### Approach: Using Shot Tracking Data
`SSQM v2.0` is heavily inspired by [AutomaticNBA's shot quality model](https://twitter.com/automaticnba/status/1734321299680305235?s=20), which uses NBA shot tracking data to categorize shots by defender distance and shot type. I'm extending AutomaticNBA's model by adding touch time as an additional category.
The categories in my model are:
1. Shot Type: `Catch and Shoot`, `Pullups`, `Less Than 10 ft`
2. Defender Distance: `0-2 Feet - Very Tight`, `2-4 Feet - Tight`, `4-6 Feet - Open`, `6+ Feet - Wide Open`
3. Touch Time: `Touch < 2 Seconds`, `Touch 2-6 Seconds`, `Touch 6+ Seconds`

### Approach: Binning Shots
There are 3 shot types, 4 defender distance and 3 touch times for the shots tracked by NBA. So we have $3\times 4\times 3=36$ shot bins. The actual number is 28 categories since Catch and shoot shots don't have touch times greater than 2 seconds. So, we have to bin the shots into 28 different categories, for each player and overall for the league. Following the same approach as `SSQM v1.0`, I use league average for each bin as the expected `FG%` or `xFG` in my model. Then determine the shot quality metrics based on the `xFG` for each bin.  
The method to calculate the shot quality metrics can be found [here](https://blog.sradjoker.cc/posts/nba-shot-quality-1/#developing-the-shot-quality-model-metrics).

## Results

Now that I have a Shot Quality Model based on defender distance, shot type, and touch time, I ran it for the 2023-24 NBA season to generate the shot quality metrics for all players. An issue here is that there are players who have not attempted many shots but made most of them. We need to filter those players out, which can be accomplished by removing players who have scored less than 100 points.

### Best Shot Makers 
Here are the best shot makers in the league i.e. the players having the highest `Shot Making` values:

|    | Player           | Team                  |   FGM |   FGA |   PTS |   xPTS |   eFG |   xeFG |   Shot_Making |   Points_Added |
|---:|:-----------------|:----------------------|------:|------:|------:|-------:|------:|-------:|--------------:|---------------:|
|  1 | Garrison Mathews | Atlanta Hawks         |    45 |    99 |   130 | 104.39 | 0.657 |  0.527 |         0.259 |           25.6 |
|  2 | Kevin Durant     | Phoenix Suns          |   491 |   911 |  1067 | 856.66 | 0.596 |  0.479 |         0.235 |          214.1 |
|  3 | Jalen Smith      | Indiana Pacers        |   159 |   251 |   356 | 301.35 | 0.721 |  0.61  |         0.221 |           55.5 |
|  4 | Matt Ryan        | New Orleans Pelicans  |    38 |    85 |   108 |  90.18 | 0.635 |  0.53  |         0.21  |           17.8 |
|  5 | Nicolas Batum    | Philadelphia 76ers    |    70 |   135 |   184 | 155.98 | 0.681 |  0.578 |         0.208 |           28.1 |
|  6 | Amir Coffey      | LA Clippers           |    90 |   161 |   215 | 181.86 | 0.668 |  0.565 |         0.206 |           33.2 |
|  7 | Sam Merrill      | Cleveland Cavaliers   |   100 |   228 |   287 | 240.32 | 0.629 |  0.527 |         0.205 |           46.7 |
|  8 | Malik Beasley    | Milwaukee Bucks       |   222 |   474 |   591 | 503.12 | 0.629 |  0.535 |         0.187 |           88.6 |
|  9 | Dante Exum       | Dallas Mavericks      |   116 |   203 |   263 | 227.43 | 0.648 |  0.56  |         0.175 |           35.5 |
| 10 | Aaron Wiggins    | Oklahoma City Thunder |   123 |   211 |   284 | 247.71 | 0.683 |  0.595 |         0.174 |           36.7 |

Most of the players in this list are movement three point shooters. An interesting observation is that 6 players on this list are common to both `SSQM v1.0` and `SSQM v2.0`. So, best shooters show up regardless of the shot quality model used. 

### Worst Shot Makers
Here are the worst shot makers in the league i.e. the players having the lowest `Shot Making` values:

|    | Player          | Team                   |   FGM |   FGA |   PTS |   xPTS |   eFG |   xeFG |   Shot_Making |   Points_Added |
|---:|:----------------|:-----------------------|------:|------:|------:|-------:|------:|-------:|--------------:|---------------:|
|  1 | Xavier Tillman  | Boston Celtics         |    87 |   210 |   186 | 251.46 | 0.443 |  0.599 |        -0.312 |          -65.5 |
|  2 | JT Thor         | Charlotte Hornets      |    51 |   139 |   119 | 153.99 | 0.434 |  0.562 |        -0.255 |          -35.4 |
|  3 | Josh Okogie     | Phoenix Suns           |    76 |   181 |   170 | 212.08 | 0.475 |  0.592 |        -0.235 |          -42.5 |
|  4 | Chris Duarte    | Sacramento Kings       |    47 |   129 |   114 | 142.15 | 0.445 |  0.555 |        -0.22  |          -28.4 |
|  5 | Isaiah Livers   | Washington Wizards     |    40 |   114 |   102 | 126.59 | 0.447 |  0.555 |        -0.216 |          -24.6 |
|  6 | Scoot Henderson | Portland Trail Blazers |   190 |   506 |   426 | 530.18 | 0.426 |  0.53  |        -0.208 |         -105.2 |
|  7 | Mike Muscala    | Detroit Pistons        |    48 |   133 |   123 | 149.01 | 0.466 |  0.564 |        -0.197 |          -26.2 |
|  8 | David Roddy     | Phoenix Suns           |   153 |   379 |   354 | 428.67 | 0.467 |  0.566 |        -0.197 |          -74.7 |
|  9 | Jaylin Williams | Oklahoma City Thunder  |    57 |   149 |   144 | 172.51 | 0.49  |  0.587 |        -0.194 |          -28.9 |
| 10 | Cam Reddish     | Los Angeles Lakers     |    80 |   199 |   191 | 227.41 | 0.48  |  0.571 |        -0.183 |          -36.4 |

9 players on this list are common to both `SSQM v1.0` and `SSQM v2.0`, but in a different order. Most of these players are end of bench players, except Scoot Henderson who is having a rough start to his NBA career.

### Most Points Added: Best Volume Shot Makers
Just looking at `Shot Making` doesn't tell the whole story of best shot makers in the league. The `Shot Making` value is actualized only if the players are taking lots of the shots. So the best shot makers in the league are who make lots of shots at high volume. To differentiate these players (who are all stars) from best players in `Shot Making` metric, I call them **Volume Shot Makers**. These players are players who have the highest `Points Added` Metric:

|    | Player                  | Team                  |   FGM |   FGA |   PTS |    xPTS |   eFG |   xeFG |   Shot_Making |   Points_Added |
|---:|:------------------------|:----------------------|------:|------:|------:|--------:|------:|-------:|--------------:|---------------:|
|  1 | Kevin Durant            | Phoenix Suns          |   491 |   911 |  1067 |  856.66 | 0.596 |  0.479 |         0.235 |          214.1 |
|  2 | Luka Doncic             | Dallas Mavericks      |   542 |  1101 |  1254 | 1102.56 | 0.573 |  0.503 |         0.138 |          151.9 |
|  3 | Shai Gilgeous-Alexander | Oklahoma City Thunder |   580 |  1063 |  1225 | 1084.6  | 0.576 |  0.51  |         0.132 |          140.3 |
|  4 | Giannis Antetokounmpo   | Milwaukee Bucks       |   626 |  1018 |  1273 | 1134.36 | 0.628 |  0.559 |         0.137 |          139.5 |
|  5 | Stephen Curry           | Golden State Warriors |   442 |   958 |  1126 |  989.61 | 0.591 |  0.519 |         0.143 |          137   |
|  6 | Devin Booker            | Phoenix Suns          |   420 |   840 |   935 |  812.23 | 0.559 |  0.486 |         0.147 |          123.5 |
|  7 | Kawhi Leonard           | LA Clippers           |   425 |   811 |   946 |  823.89 | 0.589 |  0.513 |         0.152 |          123.3 |
|  8 | Nikola Jokic            | Denver Nuggets        |   527 |   918 |  1076 |  972.83 | 0.61  |  0.551 |         0.117 |          107.4 |
|  9 | Domantas Sabonis        | Sacramento Kings      |   437 |   703 |   896 |  795.82 | 0.645 |  0.573 |         0.144 |          101.2 |
| 10 | Brandon Ingram          | New Orleans Pelicans  |   406 |   823 |   881 |  784.68 | 0.538 |  0.479 |         0.118 |           97.1 |

Similar to the previous list, 9 players on this list are common to both `SSQM v1.0` and `SSQM v2.0`, but in a different order. All os these players are considered either stars or super-stars in the NBA. Giannis rank is higher here compared to `SSQM v1.0` because the shot difficulty of his 2 pt shots which have a close defender distance and high touch time, gives him a lower `xeFG` on his shots and thus higher `Shot Making`.

### Most Points Removed: Worst Volume Shot Makers

Now let's look at the worst volume shot makers, these are players with the lowest `Points Added` metric:

|    | Player          | Team                   |   FGM |   FGA |   PTS |   xPTS |   eFG |   xeFG |   Shot_Making |   Points_Added |
|---:|:----------------|:-----------------------|------:|------:|------:|-------:|------:|-------:|--------------:|---------------:|
|  1 | Scoot Henderson | Portland Trail Blazers |   190 |   506 |   426 | 530.18 | 0.426 |  0.53  |        -0.208 |         -105.2 |
|  2 | Nikola Vucevic  | Chicago Bulls          |   369 |   786 |   794 | 895.3  | 0.509 |  0.574 |        -0.13  |         -102.2 |
|  3 | Jordan Poole    | Washington Wizards     |   292 |   729 |   685 | 773.44 | 0.472 |  0.533 |        -0.122 |          -88.9 |
|  4 | Jalen Green     | Houston Rockets        |   335 |   814 |   763 | 850.74 | 0.472 |  0.526 |        -0.108 |          -87.9 |
|  5 | Saddiq Bey      | Atlanta Hawks          |   244 |   576 |   583 | 665.95 | 0.507 |  0.579 |        -0.144 |          -82.9 |
|  6 | Josh Giddey     | Oklahoma City Thunder  |   246 |   551 |   537 | 618.71 | 0.494 |  0.57  |        -0.15  |          -82.6 |
|  7 | Jeremy Sochan   | San Antonio Spurs      |   245 |   555 |   532 | 611.38 | 0.485 |  0.557 |        -0.145 |          -80.5 |
|  8 | Jordan Clarkson | Utah Jazz              |   282 |   680 |   642 | 716.57 | 0.473 |  0.528 |        -0.11  |          -74.8 |
|  9 | David Roddy     | Phoenix Suns           |   153 |   379 |   354 | 428.67 | 0.467 |  0.566 |        -0.197 |          -74.7 |
| 10 | Ausar Thompson  | Detroit Pistons        |   197 |   406 |   407 | 480.93 | 0.501 |  0.592 |        -0.182 |          -73.9 |

7 players on this list are common to both `SSQM v1.0` and `SSQM v2.0`, but in a different order. Rookies here include Scoot and Ausar. Ausar come into the league as a bad shooter and need to improve his shooting to get more playing time. I'm a huge believer in his potential. Finally, Nikola Vucevic is having a horrible offensive season, and fits the criterion of aging centers which I discussed in the [`SSQM v1.0` blog](https://blog.sradjoker.cc/posts/nba-shot-quality-1/#worst-shot-makers).

## Conclusions
I've developed a simple shot quality model `SSQM v2.0` based on publicly available shooting data. The model is based purely on shot type, defender distance, and touch time. Then we can sort the players by the shot quality metrics like `Shot Making` and `Points Added` to find best and worst **shot makers** and **volume shot makers**.

Thank you for reading, and any feedback is appreciated. You can reach me on Twitter at [@SravanNBA](https://twitter.com/SravanNBA).









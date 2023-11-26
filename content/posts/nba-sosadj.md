+++
title = "Strength of Schedule Adjusting NBA Teams' Offensive and Defensive Ratings"
date = "2023-11-26"
description = "Using Ridge Regression to implement SRS method for adjusting offensive and defensive ratings for strength of schedule"

draft = true

[taxonomies]
tags = ["NBA","SoS","Team Ratings","Analytics"]
+++

This blog post goes through my process of adjusting NBA Teams' Offensive and Defensive Ratings for strength of schedule (SoS). This is the my first NBA post on this blog. Hopefully there will be more in the future. I'll try to update my earlier blog posts on my Medium blog [basketballobservations](https://medium.com/basketballobservations) here too. Those posts were when I was new to NBA analytics and python coding, and even blogging. I've improved a lot since and even written a few more blog posts, so hopefully the updated posts will be a lot better.

# Introduction

In NBA we measure a team's performance by using efficiency stats such as Offensive, Defensive and Net Ratings. Earlier in a NBA season, the teams play wildly different schedules. Some teams play a lot of weak teams and some teams play a lot of hard teams. Let's say a team playing weak teams is winning by a lot. Now, how do we differentiate that team from a team playing strong teams but is winning only by a little? Also, there are teams who are facing bad luck. By bad luck I mean outlier opponent shooting etc. So adjusting the team's ratings by strength of schedule and/or luck will give us a better indicator of a team's performance than raw (unadjusted) ratings.

# Motivation
This work was inspired by a tweet by [Kevin](https://twitter.com/NBACouchside). He [produced Opponent Adjusted 4-Factor ratings](https://twitter.com/NBACouchside/status/1720610641281429771) which adjust for HCA using regression of 4 Factor components with Bayesian Padding. On the same day [Krishna Narsu](https://twitter.com/knarsu3) (creator of all-in-one metric LEBRON), also released his [luck adjusted ratings](https://twitter.com/knarsu3/status/1611511553588600832) which uses both 4 factors and shot quality data. The method used by him for adjustment was [Nathan Walkers's](https://twitter.com/bbstats) LeHigh method. I tried looking into how these were done but quickly realized that I was out of my depth and have to start with something more simple. And the approach I went with is literally called simple rating system (SRS). Approximately 10 days after Krishna's and Kevin's tweets, I found another [adjusted ratings tweet](https://twitter.com/JerryEngelmann/status/1723566732038799456) using a SRS approach, this time by [Jerry](https://twitter.com/JerryEngelmann) who is the creator of ESPN's Real Plus Minus. I replied to asking how the adjustments were done, and he helped me with getting my algorithm for estimating the adjusted ratings set up.  You can find our conversation [here](https://x.com/SravanNBA/status/1724250684181348458?s=20).

# SRS Method
So, what exactly is the SRS approach? The original page on basketball-reference's website is unavailable but you can find the archived version [here](https://web.archive.org/web/20161031224357/http://www.pro-football-reference.com/blog/index4837.html). Please read that article if you want to know the details and math, because I won't do through them here. 

# Modified SRS Method
The approach used in the article, works on adjusting the net ratings for SoS but not for offensive and defensive ratings. So, Jerry suggested this approach:
> 2 rows per game, double the variables (team-off, team-def). Instead of point differential use offensive efficiency of the offensive team, in that game

Translating that to an equation form we have:

$$\hat{Team}^1_{OFF} + \hat{Team}^1_{DEF} + \hat{Team}^2_{OFF} + \hat{Team}^2_{DEF} = Team^1_{OFF} $$




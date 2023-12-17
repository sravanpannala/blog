+++
title = "Adjusting NBA Teams' Offensive and Defensive Ratings II: using HCA and Rest"
date = "2023-12-26"
description = "Adding HCA and Rest to SRS implementation"

draft = true

[taxonomies]
tags = ["NBA","SoS","Team Ratings"]
[extra]
math = true
math_auto_render = true
+++

## Introduction

This is a sequel to [my previous blog](@/posts/2023-11-26-nba-sosadj/index.md) on Adjusting NBA Ratings.

## HCA

- Kevin's method, get average HCA of all teams and adjust team net rating accordingly
- My method: do the same but for offensive and defensive ratings.

$$ ORtg^{HCA} = \bar{ORtg}^{Home} - \bar{ORtg} $$

$$ DRtg^{HCA} = \bar{DRtg}^{Home} - \bar{DRtg} $$

### HCA for 2023-24

### Multiyear HCA Trends

## Rest Advantage
- positiveresidual
- [](https://positiveresidual.com/shiny/nba/)
- time decay joke
- time decay to number of days for rest

$$ rest_i = -e^{-\Delta t_i} $$

where $\Delta t_i$ is number of days between games a and b. I sum this value for all games until that day, so that it includes effects of not only back to backs but also for situations like 2 games in 3 nights or 3 games in 5 nights.

So total rest for a particular game day and a team:

$$ rest =  \sum_{i=1}^{n} rest_i = \sum_{i=1}^{n} -e^{-\Delta t_i} $$

where `n` is number of games played by the team until that day. $\Delta t_i$ are all evaluated wrt that game day.

add e^t plot

## Rest Advantage of 2023-24
- compare with positiveresidual

## Rest Advantage of 2022-23
- compare with positiveresidual

## Multi Year Rest Trends



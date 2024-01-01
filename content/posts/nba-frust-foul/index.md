+++
title = "Estimating Frustration Fouls in the NBA"
date = "2023-12-01"
description = "Using play-by-play data to filter possessions and estimate frustration fouls"

draft = true

[taxonomies]
tags = ["play-by-play","fouls"]
categories = ["NBA"]
[extra]
toc = true
keywords = "Frustration Foul, Nikola Jokic, Transition Take Foul, play-by-play, Fastbreaks"
+++

## Introduction

One of my favorite players to watch in the NBA is Nikola Jokic of the Denver Nuggets. He is a super-intelligent player on both sides of the ball, sees angles on the floor like no-one else, and flings out many highlight-reel passes in a game. He brings an aspect of fun and joy to the game, as very few people do. The part of his game that I don’t like (probably so does Mike Malone) is his propensity to commit frustration fouls. This leads to him being in foul-trouble and playing fewer minutes because his coach doesn’t trust him to be careful with his fouls. So, I wanted to dig into how many frustration fouls Jokic commits in a season and compare it to leaders in that category in the NBA. To begin this analysis, I had to define what a frustration foul is.

## Defining Frustration Foul
Here is how I define a frustration foul:
>A frustration foul is one committed by a player within 5 seconds of them committing a turnover or missing a shot.

## Algorithm

My definition for frustration fouls and Euro-fouls are similar. For Euro-fouls, the missed shot/ turnover can be by any player on the team but the foul will be attributed to the player who commits the foul. But for frustration fouls, we only consider possessions where both the missed shot/turnover and the foul are committed by the same player.  Here is the code implemented in python to filter the necessary possessions:

```python
if isinstance(possession_event, Foul) and 
    (isinstance(possession_event.previous_event, Turnover) or not (possession_event.previous_event, FieldGoal)) 
    and possession_event.seconds_since_previous_event <= 5:
```


## Frustration Foul Leaderboard for the 2023–24 Season
Here are your leaders for frustration fouls updated as of 11/26/2023.

insert table

So, where is Jokic? I started this endeavor because of him. He is tied for 10th place with 22 other players with two frustration fouls. Now let's get back to the leaderboard. One more name I was on the lookout for was Russell Westbrook, who always seems to commit an inordinate number of frustration fouls. Predictably he is tied for third place with 4 frustration fouls. The top players in this category are Donovan Mitchell and Chris Paul. I really didn’t expect Mitchell to be here. As for Chris Paul, we know all the complaining he does during the game, and we shouldn’t be surprised to find him here.

## Frustration Foul Leaderboards for the 2020–23 Seasons
Now, let us look at the players committing the most frustration fouls over 4 seasons from 2020–23.

insert table

Finally, I found my guy Nikola Jokic who is tied for 2nd in frustration fouls at 10 fouls for the 2019–20 season. Devonte’ Graham beat him to the top spot with 11 fouls. In the 2018–19 season, James Harden lead the league in frustration fouls with 12 followed by Chris Paul with 11 who also currently leads the league in frustration fouls. Marc Gasol committed 15 frustration fouls in the 2017–18 season which lead the league by a large margin. One thing we can observe is that the number of frustration fouls committed by the league leaders is almost constant throughout the seasons. This is true for the 2019–20 season despite playing fewer games. Also, out of the 18 players shown in these leaderboards Only three Marc Gasol, Ian Mahinmi, and Nikola Jokic are big men. All the other players are guards. We can conclude that guards generally have more propensity to commit frustration fouls.

The number of frustration fouls seems to be way higher this season with Chris Paul already having 6 despite only a quarter of the season being played. The lack of preseason may contribute to this. To conclude, I have analyzed the possession data from 2017–21 and found the players who commit the most frustration fouls in those seasons. I am hoping to extend this analysis to Euro-fouls, and it would be my next blog post.

### Euro-Foul Player Leaderboards for the 2017–20 Seasons
Now, let us transition from the team leaderboards to player leaderboards. I would expect the players from the Jazz, the Clippers, the Suns, and the Hawks to show up here.

insert table

So, it seems that this list isn’t completely populated by the players from the above 4 teams. The Jazz players show up 5 times and a Clippers player shows up once. Donovan Mitchell is among the league leaders in Euro-fouls consistently for the past three years. This explains why he showed up on the Frustration foul leaderboard for the 2020–21 season. He is likely committing Euro-fouls instead of frustration fouls. Also, only two players from the above leaderboards are big men: Marc Gasol and Nikola Jokic. These two also showed up in my frustration foul leaderboards. I think the fouls committed by these two are mostly frustration fouls instead of Euro-fouls. Big men are unlikely to commit Euro-fouls as these are committed on ball-handlers who start the transition attack and big men rarely dribble the ball up the court during a fastbreak. Hence Marc Gasol and Nikola Jokic, who are matched up against these big men should almost never commit Euro-fouls.

## Comparing the leaders in Frustration Fouls and Euro Fouls
From my definition of Frustration fouls and Euro-fouls, you can see that frustration fouls are a subset of Euro fouls. We will now see which players show up in both.

Insert table

We have already talked about our favorite big men Marc Gasol and Nikola Jokic (who are European by the way) in the previous section. Only 3 other players are in the top 5 in Frustration fouls and Euro-fouls over the past three seasons. They are James Harden and Bradley Beal who are both stars who carry a disproportionate amount of their team’s offensive load. I would assume that the fouls they commit are out of pure exhaustion. The same can be said of Devonte’ Graham who was the engine of the Hornet offense as a rookie in the 2019–20 season. Notably, both Harden and Beal have been criticized by many for their lack of effort on the defensive end.

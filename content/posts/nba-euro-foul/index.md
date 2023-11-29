+++
title = "Have the New NBA Rule Changes Reduced Euro Fouls in the NBA: Foul Analysis I"
date = "2023-12-01"
description = "Using play-by-play data to filter possessions and estimate Euro fouls"
draft = true
[taxonomies]
tags = ["NBA","play-by-play","fouls"]
+++

This blog is an update to my [first](https://medium.com/basketballobservations/frustration-fouls-in-the-nba-4fed20ca0075) and [second](https://medium.com/basketballobservations/euro-fouls-in-the-nba-c8d8cee7a3a8) written almost three years ago. As mentioned in [my previous post](https://blog.sradjoker.cc/posts/nba-sosadj/), that posts were when I was new to NBA analytics and python coding, and even blogging. Updating them would hopefully convey my ideas better. Another reason for updating the post is to move it to my current blog, which is completely written on my computer and deployed automatically using github through cloudflare pages. You can actually find my blog on [github](https://github.com/sravanpannala/blog). It is written in markdown and the webpages are created using [zola](https://www.getzola.org/), giving me a fast and slick way for customizing the blog to my needs, instead of me confirming to Medium. 

## Introduction
Fastbreaks in basketball are fun, right? They produce highlight-reel dunks, posters, alley-oops, special passes, and more. It is breathtaking to see tall humans flying up the court with the ball and pulling out spectacular moves. Fastbreaks or transition basketball are also the most efficient way to score the basketball. On average, teams score more than [1.11 PPP](https://www.nba.com/stats/teams/transition?SeasonYear=2023-24&SeasonType=Regular+Season&sort=PPP&dir=1) (points per possession) in transition. So, what are defenses doing to counter this super-efficient type of offense? They are fouling intentionally early in the possession to stop the fastbreaks from even happening, i.e., nipping them in the bud.

This intentional foul is called a Euro-foul. They are named that way because the Euro-foul originated from the European game. The NBA calls Euro fouls as transition take fouls and defines them as:
> An intentional foul committed by a defender to deprive the offensive team of a fast-break opportunity

The NBA tried to [curb Euro-fouls by making changes](https://official.nba.com/example-of-clear-path-rule-simplifications/) to the [clear-path rule](https://www.washingtonpost.com/news/sports/wp/2018/08/27/the-nba-is-trying-to-speed-up-the-game-again-thats-good-for-the-league-and-for-fans/) in 2018, but it didn’t fix the issue. 
The NBA has tried to address the issue again by introducing [transition take foul rule](https://www.nba.com/news/nba-board-of-governors-approves-heightened-penalty-for-transition-take-foul) at the start of 2022-23 season by increasing the [penalty for transition take foul](https://www.sportingnews.com/us/nba/news/take-foul-transition-rule-penalties-clear-path-nba/wmsl3amguhycebagau4mmd5i) in addition to the existing clear-path rule.
Before the NBA changed the rule in 2022, the penalty for a take foul was a common foul for a defender and a side out for the offensive team. Now, in addition to the previous penalties, the offensive team is awarded one free throw, which can be taken by any player on that team. This new rule change, almost guarantees 1 point on the possession. 

This article will quantify Euro fouls prior to the 2018 rule clear path rule change, between 2018-2022 and finally after the 2022 transition take foul rule change. Then, we can see if the rule changes regarding Euro fouls are effective or not.


## Euro foul Estimation
### Definition
Let’s first get the nitty-gritty stuff out of the way. To write an algorithm for estimating Euro fouls, I have a come up with a definition of Euro foul, which can be coded:

>A Euro-foul is one committed by a player within 5 seconds of his team committing a turnover or missing a shot.

## Algorithm
To estimate Euro fouls using the above definition, I used [Darryl Blackport’s](https://twitter.com/bballport) [pbpstats API](https://pbpstats.readthedocs.io/en/latest/). It allows getting details of what happens in each possession for all the games in an NBA season. We can identify if a foul happens on the possession and if the previous possession ends in a missed shot or a turnover. We can then filter for situations when the foul occurs within 5 seconds of the previous possession and if the previous possession was a missed shot/turnover by the team. A similar algorithm can also be done to calculate the number of Euro-fouls, and this analysis will be a part of a future article. Here is the code implemented in python to filter the necessary possessions:

```python
if isinstance(possession_event, Foul) and 
    (isinstance(possession_event.previous_event, Turnover) or not (possession_event.previous_event, FieldGoal)) 
    and possession_event.seconds_since_previous_event <= 5:
    if possession_event.teamID == possession_event.previous_event.teamID:
```

As I get all the nerdy stuff out of the way, let us now look at the results.

## Euro-Fouls trends since 2012-13

## Euro-Foul Team Leaderboards for the 2020–23 Seasons

insert table

The first thing that jumps out there is that the Utah Jazz lead the league in Euro-fouls by a large margin for 2017–18 and 2018–19 seasons, and were 2nd in 2019–20 season. Their propensity for Euro-fouling was especially evident in their playoff series against the Thunder in 2018, where they ground down the Thunder offense by repeatedly Euro-fouling Russell Westbrook to prevent him from doing his devastating forays in transition. You can read an excellent article on this by Fred Katz. Also, the Utah Jazz consistently ranked in the top 5 in limiting transition opportunities over the past three seasons. Three other teams that pop up frequently on this list are the Clippers, the Suns, and the Hawks who each appear two times. From this data, I can infer that Euro-fouling is a coaching decision made by certain coaches to prevent fastbreaks. This is because the same teams show up in these lists multiple times. These coaches are willing to trade-off fouls with stopping fastbreak attacks, particularly when the other team has a superb transition attack or if they have a player who excels in transition.


## Conclusions

Thank you for reading and any feedback is appreciated. You can reach me on Twitter at [@SravanNBA](https://twitter.com/SravanNBA).
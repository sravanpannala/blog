+++
title = "How To: Adjusting NBA Teams' Offensive and Defensive Ratings using Strength of Schedule"
date = "2023-12-16"
description = "Tutorial for adjusting NBA ratings using an RAPM styled approach"

draft = true

[taxonomies]
tags = ["NBA","SoS","Team Ratings","Tutorial"]
[extra.author]
name = "Sravan"
social = "https://twitter.com/SravanNBA"
+++


This tutorial goes through my process of adjusting NBA Teams' Offensive and Defensive Ratings for strength of schedule (SoS).
First read my [**blog post**](https://blog.sradjoker.cc/posts/nba-sosadj/) on the same topic, before going any further. The blog post explains the details including the math necessary for understanding the code in this tutorial.  

The code for RAPM styled approach is adopted from [Ryan Davis' RAPM Tutorial](https://github.com/rd11490/NBA_Tutorials/tree/master/rapm) and I suggest you read that tutorial before continuing. 

If you want to run the code yourself while reading the tutorial, you can find the notebook version of this tutorial on my github:

[(https://github.com/sravanpannala/NBA-Tutorials/blob/main/sos_adjusted_ratings/how_to_adjust_nba_team_ratings_for_sos.ipynb](https://github.com/sravanpannala/NBA-Tutorials/blob/main/sos_adjusted_ratings/how_to_adjust_nba_team_ratings_for_sos.ipynb)

First let's import the necessary packages to run this code:


```python
import pandas as pd  # for processing data
import numpy as np  # for numerical operations on arrays
from tqdm import tqdm  # gives up progress bar
import time  # for time related stuff
from sklearn.linear_model import RidgeCV

# don't raise warnings when chaining pandas operations
pd.options.mode.chained_assignment = None
```

Then we will load the team information as two variable. There are 30 teams in the NBA and each team has a name and a team ID
1. `teams_list` will have a list of all team IDs
2. `teams_dict` is a dictionary mapping the team IDs to the team names.


```python
team_data = pd.read_csv("../data/NBA_teams_database.csv")
teams_list = team_data["TeamID"].tolist()
team_dict1 = team_data.to_dict(orient="records")
teams_dict = {team["TeamID"]: team["Team"] for team in team_dict1}
```

## Scraping the Data Required 
This section will cover the scraping part of the tutorial. You can skip the tutorial and go to the next section if you wish so. The data has already been scraped and is available for the 2023-24 season in the [data folder](./data/NBA_BoxScores_Adv_2023.csv).  
We will be using the `nba_api` to get the necessary data. It should be installed already if you followed the instructions in [Readme](../README.md).
The team ratings i.e. offensive, defensive and net ratings can be found for each game by using the `boxscoreadvancedv3` endpoint. This endpoint needs needs the `GameID` to get the boxscores for both teams in that game. To get `GameIDs` for all games played in the 2023-24 season, we will use the `leaguegamelog` endpoint.


```python
from nba_api.stats.endpoints import leaguegamelog, boxscoreadvancedv3

# for 2023-24 season
season = "2023"
# get the information
stats = leaguegamelog.LeagueGameLog(
    player_or_team_abbreviation="T",
    season=season,
    season_type_all_star="Regular Season",
)
# output the information as pandas dataframe
df = stats.get_data_frames()[0]
# get the GameIDs as a list
game_ids = df["GAME_ID"].tolist()
# GameIDs are repeated twich, once for home team and once for away team
# We can use numpy unique to remove the duplicates
game_ids = np.unique(game_ids)
```

Now we have a list of `game_ids` to use in `boxscoreadvancedv3` endpoint. We just put the `game_ids` in a `for` loop to get the data for each game as a dataframe. We append the generated dataframe for each game to a list of dataframes `dfa`. Finally we can use `pandas.concat` to concatenate all the dataframes into a single dataframe for the season.  
This process might take a while (10-20 minutes, depending on the number of games played), so grab a coffee or a snack and come back after some time.
There is a small (maybe big) issue, if you just run a vanilla `for` loop. The `stats.nba.com` endpoint we use to scrape the data, times out when requested too many times in a short period of time and results in a error:
```
HTTPSConnectionPool(host='stats.nba.com', port=443): Read timed out. (read timeout=30)
```
Any error will stop the `for` loop and we have to repeat again. To prevent this issue, we wrap the call to the endpoint in `try` `except` blocks and retry the endpoint for that `gameId` till it succeeds.  
I found an elegant solution for this issue while creating this tutorial which is to use the [`tenacity`](https://github.com/jd/tenacity) package.
1. We import the necessary modules from tenacity:
   1. `retry`: decorator to enable retries on the function
   2. `stop_after_attempt`: to define the maximum number of attempts. I set it as `5`
   3. `wait_fixed`: to wait for a certain amount of fixed time before retrying. The number I use is `0.6` seconds [as recommended](https://github.com/swar/nba_api/issues/176) by the authors of the `nba_api`


```python
from tenacity import retry
from tenacity.stop import stop_after_attempt
from tenacity.wait import wait_fixed
```

2. We add the `retry` decorator with the necessary options to the `get_boxscores` function, which has the `try` `except` block to handle errors


```python
@retry(stop=stop_after_attempt(5), wait=wait_fixed(0.6))
def get_boxscores(game_id):
    try:
        stats = boxscoreadvancedv3.BoxScoreAdvancedV3(game_id=game_id)
        df1 = stats.get_data_frames()[1]
    except Exception as error:
        print(error)
    return df1
```

3. Now we run the `for` loop with the decorated `get_boxscores` function. Finally, we save the scraped data as a `csv` file in the data folder. 


```python
dfa = []
for game_id in tqdm(game_ids):
    df1 = get_boxscores(game_id)
    dfa.append(df1)
df = pd.concat(dfa)
df.to_csv(f"./data/NBA_BoxScores_Adv_{season}.csv")
```

     13%|█▎        | 49/363 [00:59<06:26,  1.23s/it]

    HTTPSConnectionPool(host='stats.nba.com', port=443): Max retries exceeded with url: /stats/boxscoreadvancedv3?EndPeriod=0&EndRange=0&GameID=0022300050&RangeType=0&StartPeriod=0&StartRange=0 (Caused by ConnectTimeoutError(<urllib3.connection.HTTPSConnection object at 0x00000207782F5090>, 'Connection to stats.nba.com timed out. (connect timeout=30)'))
    

    100%|██████████| 363/363 [05:27<00:00,  1.11it/s]
    

## Loading and Pre-Processing the Data
Now lets load the data. The data has a lot of columns we don't use. So to we import only the data necessary by using the `usecols` option in `pandas.read_csv()`.


```python
season = "2023"
cols = [
    "gameId",
    "teamName",
    "teamId",
    "offensiveRating",
    "defensiveRating",
    "netRating",
    "possessions",
]
df = pd.read_csv(f"./data/NBA_BoxScores_Adv_{season}.csv", usecols=cols)
cols = ["gameId", "tId", "team", "ORtg", "DRtg", "NRtg", "poss"]
df.columns = cols
df.head(4)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gameId</th>
      <th>tId</th>
      <th>team</th>
      <th>ORtg</th>
      <th>DRtg</th>
      <th>NRtg</th>
      <th>poss</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>22300001</td>
      <td>1610612754</td>
      <td>Pacers</td>
      <td>118.6</td>
      <td>112.6</td>
      <td>6.0</td>
      <td>102.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>22300001</td>
      <td>1610612739</td>
      <td>Cavaliers</td>
      <td>112.6</td>
      <td>118.6</td>
      <td>-6.0</td>
      <td>103.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>22300002</td>
      <td>1610612749</td>
      <td>Bucks</td>
      <td>110.0</td>
      <td>104.0</td>
      <td>6.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>22300002</td>
      <td>1610612752</td>
      <td>Knicks</td>
      <td>104.0</td>
      <td>110.0</td>
      <td>-6.0</td>
      <td>101.0</td>
    </tr>
  </tbody>
</table>
</div>



As you see the printed table, each `gameId` has two entries, one of each team in the game. Each row has only the information for that team. But what we need is a combined row entry with the opponent information also.  
We will use `pandas.groupby` to achieve that. The variable to apply the operation will be `gameId`. This operation will create a `groupby` object, on which further operations can be run.


```python
df1 = df.groupby("gameId")
df1
```




    <pandas.core.groupby.generic.DataFrameGroupBy object at 0x00000207787B7710>



We then use the `nth` operation to get the 1st and 2nd rows of each game.



```python
df1_1 = df1.nth(0)
df1_2 = df1.nth(1)
display(df1_1.head(2))
display(df1_2.head(2))
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gameId</th>
      <th>tId</th>
      <th>team</th>
      <th>ORtg</th>
      <th>DRtg</th>
      <th>NRtg</th>
      <th>poss</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>22300001</td>
      <td>1610612754</td>
      <td>Pacers</td>
      <td>118.6</td>
      <td>112.6</td>
      <td>6.0</td>
      <td>102.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>22300002</td>
      <td>1610612749</td>
      <td>Bucks</td>
      <td>110.0</td>
      <td>104.0</td>
      <td>6.0</td>
      <td>100.0</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gameId</th>
      <th>tId</th>
      <th>team</th>
      <th>ORtg</th>
      <th>DRtg</th>
      <th>NRtg</th>
      <th>poss</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>22300001</td>
      <td>1610612739</td>
      <td>Cavaliers</td>
      <td>112.6</td>
      <td>118.6</td>
      <td>-6.0</td>
      <td>103.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>22300002</td>
      <td>1610612752</td>
      <td>Knicks</td>
      <td>104.0</td>
      <td>110.0</td>
      <td>-6.0</td>
      <td>101.0</td>
    </tr>
  </tbody>
</table>
</div>


We can then rename the columns of the 1st dataframe, adding `1` to all its column names, except the `gameId` column (which is needed for the merging operation later). For the 2nd dataframe, similarly add `2` to the columns names.


```python
df1_1.columns = ["gameId"] + [s + "1" for s in df1_1.columns if s != "gameId"]
df1_2.columns = ["gameId"] + [s + "2" for s in df1_2.columns if s != "gameId"]
display(df1_1.head(2))
display(df1_2.head(2))
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gameId</th>
      <th>tId1</th>
      <th>team1</th>
      <th>ORtg1</th>
      <th>DRtg1</th>
      <th>NRtg1</th>
      <th>poss1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>22300001</td>
      <td>1610612754</td>
      <td>Pacers</td>
      <td>118.6</td>
      <td>112.6</td>
      <td>6.0</td>
      <td>102.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>22300002</td>
      <td>1610612749</td>
      <td>Bucks</td>
      <td>110.0</td>
      <td>104.0</td>
      <td>6.0</td>
      <td>100.0</td>
    </tr>
  </tbody>
</table>
</div>



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gameId</th>
      <th>tId2</th>
      <th>team2</th>
      <th>ORtg2</th>
      <th>DRtg2</th>
      <th>NRtg2</th>
      <th>poss2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>22300001</td>
      <td>1610612739</td>
      <td>Cavaliers</td>
      <td>112.6</td>
      <td>118.6</td>
      <td>-6.0</td>
      <td>103.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>22300002</td>
      <td>1610612752</td>
      <td>Knicks</td>
      <td>104.0</td>
      <td>110.0</td>
      <td>-6.0</td>
      <td>101.0</td>
    </tr>
  </tbody>
</table>
</div>


We then merge the two dataframes `df1_1` and `df1_2` on the column `gameId`, generating the dataframe we need.


```python
df1_3 = pd.merge(df1_1, df1_2, on="gameId")
display(df1_3.head(2))
```


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gameId</th>
      <th>tId1</th>
      <th>team1</th>
      <th>ORtg1</th>
      <th>DRtg1</th>
      <th>NRtg1</th>
      <th>poss1</th>
      <th>tId2</th>
      <th>team2</th>
      <th>ORtg2</th>
      <th>DRtg2</th>
      <th>NRtg2</th>
      <th>poss2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>22300001</td>
      <td>1610612754</td>
      <td>Pacers</td>
      <td>118.6</td>
      <td>112.6</td>
      <td>6.0</td>
      <td>102.0</td>
      <td>1610612739</td>
      <td>Cavaliers</td>
      <td>112.6</td>
      <td>118.6</td>
      <td>-6.0</td>
      <td>103.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>22300002</td>
      <td>1610612749</td>
      <td>Bucks</td>
      <td>110.0</td>
      <td>104.0</td>
      <td>6.0</td>
      <td>100.0</td>
      <td>1610612752</td>
      <td>Knicks</td>
      <td>104.0</td>
      <td>110.0</td>
      <td>-6.0</td>
      <td>101.0</td>
    </tr>
  </tbody>
</table>
</div>


One more step remaining. What we have right now is one row of each game. But, what we need is two rows for each game as described in my [blog post](https://blog.sradjoker.cc/posts/nba-sosadj/#modified-srs-method). To get that dataframe, we repeat the process above, with `0` and `1` flipped when performing the `nth` operation. Finally we merge the two dataframes `df1_3` and `df1_6`, to get the combined dataframe with two rows for each game.


```python
df1_4 = df1.nth(1)
df1_5 = df1.nth(0)
df1_4.columns = ["gameId"] + [s + "1" for s in df1_4.columns if s != "gameId"]
df1_5.columns = ["gameId"] + [s + "2" for s in df1_5.columns if s != "gameId"]
df1_6 = pd.merge(df1_4, df1_5, on="gameId")
df2 = pd.concat([df1_3, df1_6]).sort_values(by="gameId").reset_index(drop=True)
data = df2.copy()
data.head(4)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gameId</th>
      <th>tId1</th>
      <th>team1</th>
      <th>ORtg1</th>
      <th>DRtg1</th>
      <th>NRtg1</th>
      <th>poss1</th>
      <th>tId2</th>
      <th>team2</th>
      <th>ORtg2</th>
      <th>DRtg2</th>
      <th>NRtg2</th>
      <th>poss2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>22300001</td>
      <td>1610612754</td>
      <td>Pacers</td>
      <td>118.6</td>
      <td>112.6</td>
      <td>6.0</td>
      <td>102.0</td>
      <td>1610612739</td>
      <td>Cavaliers</td>
      <td>112.6</td>
      <td>118.6</td>
      <td>-6.0</td>
      <td>103.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>22300001</td>
      <td>1610612739</td>
      <td>Cavaliers</td>
      <td>112.6</td>
      <td>118.6</td>
      <td>-6.0</td>
      <td>103.0</td>
      <td>1610612754</td>
      <td>Pacers</td>
      <td>118.6</td>
      <td>112.6</td>
      <td>6.0</td>
      <td>102.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>22300002</td>
      <td>1610612752</td>
      <td>Knicks</td>
      <td>104.0</td>
      <td>110.0</td>
      <td>-6.0</td>
      <td>101.0</td>
      <td>1610612749</td>
      <td>Bucks</td>
      <td>110.0</td>
      <td>104.0</td>
      <td>6.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>22300002</td>
      <td>1610612749</td>
      <td>Bucks</td>
      <td>110.0</td>
      <td>104.0</td>
      <td>6.0</td>
      <td>100.0</td>
      <td>1610612752</td>
      <td>Knicks</td>
      <td>104.0</td>
      <td>110.0</td>
      <td>-6.0</td>
      <td>101.0</td>
    </tr>
  </tbody>
</table>
</div>



# Processing the Data
To process the data in a format required by the Ridge Regression algorithm `RidgeCV`, we define the following functions:
## maps_teams()
1. Makes the matrix rows to be used in ridge regression
2. The weights for each team = 1/2
3. Equations per game are:  
$$\frac{1}{2}\hat{Team}^1_{OFF} + \frac{1}{2}\hat{Team}^2_{DEF} = Team^1_{OFF} $$
$$\frac{1}{2}\hat{Team}^2_{OFF} + \frac{1}{2}\hat{Team}^1_{DEF} = Team^2_{OFF} $$
4. The reason for doing this is that for unadjusted values of a game:
$$ Team^1_{OFF} = Team^2_{DEF} $$  
5. So,
$$ Team^1_{OFF} = 0.5\times Team^1_{OFF} + 0.5\times Team^2_{DEF} $$
6. Therefore I use a similar structure for estimating adjusted ratings





```python
def map_teams(row_in, teams, scale):
    t1 = row_in[0]
    t2 = row_in[1]

    rowOut = np.zeros([len(teams) * 2])
    rowOut[teams.index(t1)] = scale
    rowOut[teams.index(t2) + len(teams)] = scale

    return rowOut
```

## convert_to_matrices()
1. Converts each row of data dataframe to x stints.
2. Then maps those rows using `map_teams` function to get matrix X rows
3. Gets Y rows. Here Y is `ORtg1` i.e. we are trying to predict the offensive rating of the 1st team for every row


```python
def convert_to_matricies(possessions, name, teams, scale=1):
    # extract only the columns we need
    # Convert the columns of player ids into a numpy matrix
    stints_x_base = possessions[["tId1", "tId2"]].to_numpy()
    # Apply our mapping function to the numpy matrix
    stint_X_rows = np.apply_along_axis(map_teams, 1, stints_x_base, teams, scale=scale)
    # Convert the column of target values into a numpy matrix
    stint_Y_rows = possessions[name].to_numpy()

    # return matricies and possessions series
    return stint_X_rows, stint_Y_rows
```

## lambda_to_alpha()
- In stats world (`R`), `glmnet()` is used for Ridge Regression and uses the parameter \\(\lambda\\). Most the NBA stats people use this parameter \\(\lambda\\) for discussing the regularization parameter. But `sklearn.linear_model.RidgeCV()` has a parameter \\(\alpha\\), which isn't the same. 
- So we need to convert \\(\lambda\\) to \\(\alpha\\) needed for Ridge CV. [More details here](https://stats.stackexchange.com/questions/160096/what-are-the-differences-between-ridge-regression-using-rs-glmnet-and-pythons)


```python
def lambda_to_alpha(lambda_value, samples):
    return (lambda_value * samples) / 2.0
```

## calculate_netrtg()
1. Converts lambdas to alphas using `lambda_to_alpha` function
2. Defines the ridge regression problem using `scikit-learn`'s `RidgeCV` algorithm
3. `cv=5` is chosen i.e. k-fold cross-validation splitting strategy using `k=5`
4. `Intercept` is set as true. This value is to be added later to our estimation results to get Offensive and Defensive ratings.
5. Gets coefficients and intercept
6. Add intercept to intercept to get adjusted ratings. Use adjusted off and def ratings to calculate adjusted net rating.
7. Create and return adjusted ratings dataframe


```python
def calculate_netrtg(train_x, train_y, lambdas, teams_list):
    alphas = [lambda_to_alpha(l, train_x.shape[0]) for l in lambdas]
    # create a 5 fold CV ridgeCV model. Our target data is not centered at 0, so we want to fit to an intercept.
    clf = RidgeCV(alphas=alphas, cv=5, fit_intercept=True)

    # fit our training data
    model = clf.fit(
        train_x,
        train_y,
    )

    # convert our list of players into a mx1 matrix
    team_arr = np.transpose(np.array(teams_list).reshape(1, len(teams_list)))

    # extract our coefficients into the offensive and defensive parts
    coef_offensive_array = model.coef_[0 : len(teams_list)][np.newaxis].T
    coef_defensive_array = model.coef_[len(teams_list) : 2 * len(teams_list)][
        np.newaxis
    ].T
    # concatenate the offensive and defensive values with the playey ids into a mx3 matrix
    team_id_with_coef = np.concatenate(
        [team_arr, coef_offensive_array, coef_defensive_array], axis=1
    )
    # build a dataframe from our matrix
    teams_coef = pd.DataFrame(team_id_with_coef)
    intercept = model.intercept_
    teams_coef.columns = ["tId", "aOFF", "aDEF"]
    teams_coef["aNET"] = teams_coef["aOFF"] - teams_coef["aDEF"]
    teams_coef["aOFF"] = teams_coef["aOFF"] + intercept
    teams_coef["aDEF"] = teams_coef["aDEF"] + intercept
    teams_coef["Team"] = teams_coef["tId"].map(teams_dict)
    results = teams_coef[["tId", "Team", "aOFF", "aDEF", "aNET"]]
    results = results.sort_values(by=["aNET"], ascending=False).reset_index(drop=True)
    return results, model, intercept
```

# Estimating Adjusted Ratings
Next, we run the functions defined above to generated the adjusted ratings


```python
train_x, train_y = convert_to_matricies(data, "ORtg1", teams_list, scale=0.5)
lambdas_net = [0.015, 0.075, 0.15]
results_adj, model, intercept = calculate_netrtg(
    train_x, train_y, lambdas_net, teams_list
)
print(f"Intercept = {intercept}")
```

    Intercept = 114.2197043446658
    

The intercept here can be interpreted as the league average offensive/defensive rating.
Here are the adjusted ratings.


```python
results_adj
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tId</th>
      <th>Team</th>
      <th>aOFF</th>
      <th>aDEF</th>
      <th>aNET</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.610613e+09</td>
      <td>Philadelphia 76ers</td>
      <td>121.065207</td>
      <td>110.772873</td>
      <td>10.292335</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.610613e+09</td>
      <td>Boston Celtics</td>
      <td>118.828331</td>
      <td>108.764236</td>
      <td>10.064095</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.610613e+09</td>
      <td>Oklahoma City Thunder</td>
      <td>117.702690</td>
      <td>110.814878</td>
      <td>6.887812</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1.610613e+09</td>
      <td>Minnesota Timberwolves</td>
      <td>113.207243</td>
      <td>106.628440</td>
      <td>6.578803</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.610613e+09</td>
      <td>Denver Nuggets</td>
      <td>118.395144</td>
      <td>113.090987</td>
      <td>5.304157</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1.610613e+09</td>
      <td>LA Clippers</td>
      <td>115.366218</td>
      <td>111.064890</td>
      <td>4.301329</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1.610613e+09</td>
      <td>Orlando Magic</td>
      <td>113.446035</td>
      <td>109.345141</td>
      <td>4.100894</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1.610613e+09</td>
      <td>New York Knicks</td>
      <td>117.214095</td>
      <td>113.291210</td>
      <td>3.922885</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1.610613e+09</td>
      <td>Houston Rockets</td>
      <td>111.781191</td>
      <td>107.967128</td>
      <td>3.814063</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1.610613e+09</td>
      <td>Milwaukee Bucks</td>
      <td>118.657846</td>
      <td>115.338466</td>
      <td>3.319381</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1.610613e+09</td>
      <td>Brooklyn Nets</td>
      <td>116.937071</td>
      <td>114.575413</td>
      <td>2.361658</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1.610613e+09</td>
      <td>Indiana Pacers</td>
      <td>122.514626</td>
      <td>120.553872</td>
      <td>1.960754</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1.610613e+09</td>
      <td>Dallas Mavericks</td>
      <td>118.932015</td>
      <td>117.355888</td>
      <td>1.576127</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1.610613e+09</td>
      <td>New Orleans Pelicans</td>
      <td>114.101714</td>
      <td>113.092424</td>
      <td>1.009290</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1.610613e+09</td>
      <td>Golden State Warriors</td>
      <td>114.940190</td>
      <td>114.182474</td>
      <td>0.757716</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1.610613e+09</td>
      <td>Miami Heat</td>
      <td>114.132399</td>
      <td>113.518409</td>
      <td>0.613991</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1.610613e+09</td>
      <td>Atlanta Hawks</td>
      <td>118.941745</td>
      <td>118.485097</td>
      <td>0.456648</td>
    </tr>
    <tr>
      <th>17</th>
      <td>1.610613e+09</td>
      <td>Phoenix Suns</td>
      <td>116.960966</td>
      <td>116.528575</td>
      <td>0.432391</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1.610613e+09</td>
      <td>Cleveland Cavaliers</td>
      <td>111.023001</td>
      <td>110.911526</td>
      <td>0.111475</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1.610613e+09</td>
      <td>Los Angeles Lakers</td>
      <td>112.332601</td>
      <td>112.222641</td>
      <td>0.109959</td>
    </tr>
    <tr>
      <th>20</th>
      <td>1.610613e+09</td>
      <td>Sacramento Kings</td>
      <td>115.306403</td>
      <td>115.211154</td>
      <td>0.095249</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1.610613e+09</td>
      <td>Toronto Raptors</td>
      <td>112.389538</td>
      <td>114.521311</td>
      <td>-2.131772</td>
    </tr>
    <tr>
      <th>22</th>
      <td>1.610613e+09</td>
      <td>Chicago Bulls</td>
      <td>111.271702</td>
      <td>115.736148</td>
      <td>-4.464446</td>
    </tr>
    <tr>
      <th>23</th>
      <td>1.610613e+09</td>
      <td>Memphis Grizzlies</td>
      <td>106.322251</td>
      <td>113.173911</td>
      <td>-6.851659</td>
    </tr>
    <tr>
      <th>24</th>
      <td>1.610613e+09</td>
      <td>Portland Trail Blazers</td>
      <td>106.938770</td>
      <td>114.757369</td>
      <td>-7.818600</td>
    </tr>
    <tr>
      <th>25</th>
      <td>1.610613e+09</td>
      <td>Charlotte Hornets</td>
      <td>112.456339</td>
      <td>120.395203</td>
      <td>-7.938864</td>
    </tr>
    <tr>
      <th>26</th>
      <td>1.610613e+09</td>
      <td>Utah Jazz</td>
      <td>110.533878</td>
      <td>118.914809</td>
      <td>-8.380931</td>
    </tr>
    <tr>
      <th>27</th>
      <td>1.610613e+09</td>
      <td>Washington Wizards</td>
      <td>111.230457</td>
      <td>120.661742</td>
      <td>-9.431285</td>
    </tr>
    <tr>
      <th>28</th>
      <td>1.610613e+09</td>
      <td>San Antonio Spurs</td>
      <td>107.330475</td>
      <td>117.157667</td>
      <td>-9.827192</td>
    </tr>
    <tr>
      <th>29</th>
      <td>1.610613e+09</td>
      <td>Detroit Pistons</td>
      <td>106.330988</td>
      <td>117.557249</td>
      <td>-11.226261</td>
    </tr>
  </tbody>
</table>
</div>



# Finishing Touches
We're not done yet. Now we need to compare the adjusted ratings with the unadjusted ones. But, we haven't calculated the unadjusted ratings yet. Let's do it now.

For a single game:
$$ PTS_{OFF}*100 = ORtg^1 \times poss^1 $$
$$ PTS_{DEF}*100 = DRtg^1 \times poss^1 $$

Applying these operations on the `data` dataframe:


```python
data["pts_off"] = data["ORtg1"] * data["poss1"]
data["pts_def"] = data["DRtg1"] * data["poss1"]
```

We have to use the `groupby` operation again, now on the `tId1` column. After the `groupby` operation, we chain an `agg` (aggregate) operation, which applies a function on all rows of the group. The function we chose here is `sum`, which adds all the `pts` and and `poss` for a team.


```python
off_p = data.groupby(["tId1"])[["poss1", "pts_off"]].agg("sum").reset_index()
def_p = data.groupby(["tId1"])[["poss1", "pts_def"]].agg("sum").reset_index()
```

The unadjusted team ratings would then be:
$$ OFF = \frac{PTS_{OFF}^{Total}}{poss^{Total}} $$ 
$$ DEF = \frac{PTS_{DEF}^{Total}}{poss^{Total}} $$ 


```python
off_p["OFF"] = off_p["pts_off"] / off_p["poss1"]
off_p = off_p[["tId1", "OFF"]]
def_p["DEF"] = def_p["pts_def"] / def_p["poss1"]
def_p = def_p[["tId1", "DEF"]]
```

We then merge these ratings to the `results_adj` dataframe


```python
results_net = pd.merge(off_p, def_p, on=["tId1"])
results_net["NET"] = results_net["OFF"] - results_net["DEF"]
results_net.rename(columns={"tId1": "tId"}, inplace=True)
results_net = results_net.astype(float).round(2)
results_net["tId"] = results_net["tId"].astype(int)
results_adj["tId"] = results_adj["tId"].astype(int)
results_comb = pd.merge(results_net, results_adj, on=["tId"])
results_comb["aOFF"] = results_comb["aOFF"]
results_comb["aDEF"] = results_comb["aDEF"]
results_comb["oSOS"] = results_comb["aOFF"] - results_comb["OFF"]
results_comb["dSOS"] = results_comb["DEF"] - results_comb["aDEF"]
results_comb["SOS"] = results_comb["oSOS"] + results_comb["dSOS"]
results_comb.iloc[:, 1:] = results_comb.iloc[:, 1:].round(1)
results = results_comb[
    ["Team", "OFF", "oSOS", "aOFF", "DEF", "dSOS", "aDEF", "NET", "SOS", "aNET"]
]
results = results.sort_values(by="aNET", ascending=0).reset_index(drop=True)
results.index = results.index + 1
```

## Reminder
You can find the notebook version of this tutorial on my github:
[(https://github.com/sravanpannala/NBA-Tutorials/blob/main/sos_adjusted_ratings/how_to_adjust_nba_team_ratings_for_sos.ipynb](https://github.com/sravanpannala/NBA-Tutorials/blob/main/sos_adjusted_ratings/how_to_adjust_nba_team_ratings_for_sos.ipynb)

## Final Combined Data table:
You can save it as `csv` file and then you some fancy visualization tool to create a [pretty looking table](https://twitter.com/SravanNBA/status/1725722980159045792) and/or [efficiency landscape graph](https://twitter.com/SravanNBA/status/1727377558176661774)


```python
results
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Team</th>
      <th>OFF</th>
      <th>oSOS</th>
      <th>aOFF</th>
      <th>DEF</th>
      <th>dSOS</th>
      <th>aDEF</th>
      <th>NET</th>
      <th>SOS</th>
      <th>aNET</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Philadelphia 76ers</td>
      <td>121.2</td>
      <td>-0.1</td>
      <td>121.1</td>
      <td>110.9</td>
      <td>0.2</td>
      <td>110.8</td>
      <td>10.3</td>
      <td>0.0</td>
      <td>10.3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Boston Celtics</td>
      <td>118.3</td>
      <td>0.5</td>
      <td>118.8</td>
      <td>109.6</td>
      <td>0.9</td>
      <td>108.8</td>
      <td>8.7</td>
      <td>1.4</td>
      <td>10.1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Oklahoma City Thunder</td>
      <td>117.6</td>
      <td>0.1</td>
      <td>117.7</td>
      <td>110.6</td>
      <td>-0.2</td>
      <td>110.8</td>
      <td>7.0</td>
      <td>-0.1</td>
      <td>6.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Minnesota Timberwolves</td>
      <td>113.3</td>
      <td>-0.1</td>
      <td>113.2</td>
      <td>106.6</td>
      <td>-0.1</td>
      <td>106.6</td>
      <td>6.7</td>
      <td>-0.2</td>
      <td>6.6</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Denver Nuggets</td>
      <td>117.3</td>
      <td>1.1</td>
      <td>118.4</td>
      <td>112.6</td>
      <td>-0.5</td>
      <td>113.1</td>
      <td>4.7</td>
      <td>0.6</td>
      <td>5.3</td>
    </tr>
    <tr>
      <th>6</th>
      <td>LA Clippers</td>
      <td>115.4</td>
      <td>-0.1</td>
      <td>115.4</td>
      <td>110.6</td>
      <td>-0.5</td>
      <td>111.1</td>
      <td>4.8</td>
      <td>-0.5</td>
      <td>4.3</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Orlando Magic</td>
      <td>113.8</td>
      <td>-0.4</td>
      <td>113.4</td>
      <td>109.5</td>
      <td>0.2</td>
      <td>109.3</td>
      <td>4.3</td>
      <td>-0.2</td>
      <td>4.1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>New York Knicks</td>
      <td>117.3</td>
      <td>-0.1</td>
      <td>117.2</td>
      <td>113.3</td>
      <td>0.0</td>
      <td>113.3</td>
      <td>4.0</td>
      <td>-0.1</td>
      <td>3.9</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Houston Rockets</td>
      <td>111.4</td>
      <td>0.3</td>
      <td>111.8</td>
      <td>107.4</td>
      <td>-0.5</td>
      <td>108.0</td>
      <td>4.0</td>
      <td>-0.2</td>
      <td>3.8</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Milwaukee Bucks</td>
      <td>119.3</td>
      <td>-0.7</td>
      <td>118.7</td>
      <td>115.7</td>
      <td>0.4</td>
      <td>115.3</td>
      <td>3.6</td>
      <td>-0.3</td>
      <td>3.3</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Brooklyn Nets</td>
      <td>116.9</td>
      <td>0.0</td>
      <td>116.9</td>
      <td>115.0</td>
      <td>0.4</td>
      <td>114.6</td>
      <td>2.0</td>
      <td>0.4</td>
      <td>2.4</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Indiana Pacers</td>
      <td>122.4</td>
      <td>0.1</td>
      <td>122.5</td>
      <td>120.1</td>
      <td>-0.4</td>
      <td>120.6</td>
      <td>2.3</td>
      <td>-0.3</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Dallas Mavericks</td>
      <td>118.5</td>
      <td>0.4</td>
      <td>118.9</td>
      <td>116.4</td>
      <td>-1.0</td>
      <td>117.4</td>
      <td>2.2</td>
      <td>-0.6</td>
      <td>1.6</td>
    </tr>
    <tr>
      <th>14</th>
      <td>New Orleans Pelicans</td>
      <td>114.1</td>
      <td>0.0</td>
      <td>114.1</td>
      <td>113.1</td>
      <td>-0.0</td>
      <td>113.1</td>
      <td>1.0</td>
      <td>-0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Golden State Warriors</td>
      <td>114.0</td>
      <td>1.0</td>
      <td>114.9</td>
      <td>114.2</td>
      <td>0.0</td>
      <td>114.2</td>
      <td>-0.2</td>
      <td>1.0</td>
      <td>0.8</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Miami Heat</td>
      <td>114.7</td>
      <td>-0.6</td>
      <td>114.1</td>
      <td>113.5</td>
      <td>-0.0</td>
      <td>113.5</td>
      <td>1.2</td>
      <td>-0.6</td>
      <td>0.6</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Atlanta Hawks</td>
      <td>118.7</td>
      <td>0.2</td>
      <td>118.9</td>
      <td>118.8</td>
      <td>0.4</td>
      <td>118.5</td>
      <td>-0.1</td>
      <td>0.6</td>
      <td>0.5</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Phoenix Suns</td>
      <td>116.6</td>
      <td>0.4</td>
      <td>117.0</td>
      <td>115.4</td>
      <td>-1.1</td>
      <td>116.5</td>
      <td>1.2</td>
      <td>-0.7</td>
      <td>0.4</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Los Angeles Lakers</td>
      <td>112.2</td>
      <td>0.1</td>
      <td>112.3</td>
      <td>111.8</td>
      <td>-0.5</td>
      <td>112.2</td>
      <td>0.4</td>
      <td>-0.3</td>
      <td>0.1</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Sacramento Kings</td>
      <td>114.6</td>
      <td>0.7</td>
      <td>115.3</td>
      <td>114.9</td>
      <td>-0.3</td>
      <td>115.2</td>
      <td>-0.2</td>
      <td>0.3</td>
      <td>0.1</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Cleveland Cavaliers</td>
      <td>111.1</td>
      <td>-0.0</td>
      <td>111.0</td>
      <td>111.5</td>
      <td>0.6</td>
      <td>110.9</td>
      <td>-0.5</td>
      <td>0.6</td>
      <td>0.1</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Toronto Raptors</td>
      <td>112.7</td>
      <td>-0.3</td>
      <td>112.4</td>
      <td>114.9</td>
      <td>0.4</td>
      <td>114.5</td>
      <td>-2.2</td>
      <td>0.1</td>
      <td>-2.1</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Chicago Bulls</td>
      <td>111.6</td>
      <td>-0.3</td>
      <td>111.3</td>
      <td>115.7</td>
      <td>-0.1</td>
      <td>115.7</td>
      <td>-4.1</td>
      <td>-0.4</td>
      <td>-4.5</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Memphis Grizzlies</td>
      <td>106.5</td>
      <td>-0.2</td>
      <td>106.3</td>
      <td>112.7</td>
      <td>-0.5</td>
      <td>113.2</td>
      <td>-6.2</td>
      <td>-0.7</td>
      <td>-6.9</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Portland Trail Blazers</td>
      <td>107.2</td>
      <td>-0.3</td>
      <td>106.9</td>
      <td>114.2</td>
      <td>-0.5</td>
      <td>114.8</td>
      <td>-7.0</td>
      <td>-0.8</td>
      <td>-7.8</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Charlotte Hornets</td>
      <td>112.6</td>
      <td>-0.1</td>
      <td>112.5</td>
      <td>120.4</td>
      <td>-0.0</td>
      <td>120.4</td>
      <td>-7.8</td>
      <td>-0.2</td>
      <td>-7.9</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Utah Jazz</td>
      <td>110.4</td>
      <td>0.1</td>
      <td>110.5</td>
      <td>118.1</td>
      <td>-0.8</td>
      <td>118.9</td>
      <td>-7.6</td>
      <td>-0.7</td>
      <td>-8.4</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Washington Wizards</td>
      <td>111.8</td>
      <td>-0.6</td>
      <td>111.2</td>
      <td>121.4</td>
      <td>0.7</td>
      <td>120.7</td>
      <td>-9.6</td>
      <td>0.1</td>
      <td>-9.4</td>
    </tr>
    <tr>
      <th>29</th>
      <td>San Antonio Spurs</td>
      <td>107.0</td>
      <td>0.3</td>
      <td>107.3</td>
      <td>117.3</td>
      <td>0.1</td>
      <td>117.2</td>
      <td>-10.3</td>
      <td>0.5</td>
      <td>-9.8</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Detroit Pistons</td>
      <td>106.8</td>
      <td>-0.5</td>
      <td>106.3</td>
      <td>117.8</td>
      <td>0.2</td>
      <td>117.6</td>
      <td>-11.0</td>
      <td>-0.2</td>
      <td>-11.2</td>
    </tr>
  </tbody>
</table>
</div>

<!-- README.md is generated from README.Rmd. Please edit that file -->

# nflseedR <img src='man/figures/logo.png' align="right" height="139" />

<!-- badges: start -->
<!-- badges: end -->

## Motivation

The goal of nflseedR is to allow NFL modelers to simulate NFL seasons
using the model, by taking the hard work of navigating the complex rules
for division ranking, playoff seeding, and draft order off your plate
and let you focus on your model. This can also aid in sports betting,
such as betting on futures or win totals.

The package can run thousands of simulations of the NFL regular season,
based on a model you input. Within each simulated season, it will
calculate the division standings and playoff seedings for you. It will
also the generate the playoff games and simulate these as well, and
calculate the order for next year’s NFL draft. These can be used to
examine the probability of team making the playoffs or winning the Super
Bowl, based on your model.

The season simulations will take all completed games into account
already, and only simulate from there forward, including if run during
the playoffs. It can also be run as a fresh season, wiping away results
and simulating from scratch.

The season simulation code for nflseedR was developed by Lee Sharpe
([@LeeSharpeNFL](<https://twitter.com/leesharpenfl>)) and building it
as package was developed by Sebastian Carl
([@mrcaseb](<https://twitter.com/mrcaseb>)).

## Installation

<!-- You can install the released version of nflseedR from [CRAN](https://CRAN.R-project.org) with: -->
<!-- ``` r -->
<!-- install.packages("nflseedR") -->
<!-- ``` -->

You can install nflseedR from [GitHub](https://github.com/) with:

``` r
if (!requireNamespace("remotes", quietly = TRUE)) {install.packages("remotes")}
remotes::install_github("leesharpe/nflseedR")
```

## Run a Quick Simulation

Let's do a quick simualation! We'll apply the `fresh_season = TRUE` option,
which blanks out all of the game results and simulates everything from scratch.
This is a Monte Carlo style of simulation, meaning that there are seasons
simulated.

By default, 1000 simulations are done. 

``` r
> library(nflseedR)
> sims <- simulate_nfl(fresh_season = TRUE)
* 2021-02-13 13:54:04: Loading games data
* 2021-02-13 13:54:22: Beginning simulation of 1000 seasons in 1 rounds
* 2021-02-13 13:56:22: Combining simulation data
* 2021-02-13 13:56:22: Aggregating across simulations
```

The output contains a lot of pre-aggregated information, as well as the individual
results from each game of each simulation. For example, let's look at the overall
results for the Bears:

*Note: This is example randomized output, your output will differ.*

``` r
> library(tidyverse)
> sims$overall %>% filter(team == "CHI")
# A tibble: 1 x 11
  conf  division  team   wins playoff  div1 seed1 won_conf won_sb draft1 draft5
  <chr> <chr>     <chr> <dbl>   <dbl> <dbl> <dbl>    <dbl>  <dbl>  <dbl>  <dbl>
1 NFC   NFC North CHI    10.6   0.696 0.301 0.076    0.082  0.042      0  0.002
```

We can see the Bears got 10.6 wins on average. They made the playoffs 70%
of the time, and won the division 30% of the time. They won the Super Bowl in 4%,
and have no realistic shot at the #1 or even a top 5 draft pick. The `teams`
section of the output will show how a team did in each simulated season. 

``` r
> sims$teams %>% filter(team == "CHI")
# A tibble: 1,000 x 16
     sim team  conf  division  games  wins true_wins win_pct div_pct conf_pct   sov   sos div_rank  seed  exit draft_order
   <dbl> <chr> <chr> <chr>     <int> <dbl>     <int>   <dbl>   <dbl>    <dbl> <dbl> <dbl>    <dbl> <dbl> <dbl>       <dbl>
 1     1 CHI   NFC   NFC North    16    14        14   0.875   1        0.833 0.491 0.496        1     1    22          32
 2     2 CHI   NFC   NFC North    16    12        12   0.75    0.667    0.667 0.438 0.508        2     5    19          27
 3     3 CHI   NFC   NFC North    16     9         9   0.562   0.667    0.5   0.396 0.531        2    NA    17          18
 4     4 CHI   NFC   NFC North    16     9         9   0.562   1        0.667 0.444 0.512        2     6    18          23
 5     5 CHI   NFC   NFC North    16     9         9   0.562   0.5      0.5   0.431 0.535        3    NA    17          18
 6     6 CHI   NFC   NFC North    16    10        10   0.625   0.833    0.583 0.381 0.531        2     7    18          23
 7     7 CHI   NFC   NFC North    16    12        12   0.75    0.667    0.667 0.432 0.531        2     5    18          23
 8     8 CHI   NFC   NFC North    16    13        13   0.812   1        0.833 0.476 0.527        1     3    19          28
 9     9 CHI   NFC   NFC North    16     8         8   0.5     0.5      0.583 0.469 0.531        3     7    18          19
10    10 CHI   NFC   NFC North    16    10        10   0.625   0.5      0.667 0.388 0.531        3     6    18          22
```

Oh hey, the Bears got the 32nd draft pick and Week "22" exit, which both mean they
won the Super Bowl! Let's check the individual games and see how that unfolded.

``` r
> sims$games %>% filter(sim == 1, game_type != "REG")
# A tibble: 13 x 9
     sim game_type  week away_team home_team away_rest home_rest location result
   <dbl> <chr>     <int> <chr>     <chr>         <dbl>     <dbl> <chr>     <int>
 1     1 WC           18 MIA       JAX               7         7 Home        -51
 2     1 WC           18 PIT       LAC               7         7 Home         -4
 3     1 WC           18 BAL       CIN               7         7 Home        -19
 4     1 WC           18 SF        ARI               7         7 Home         13
 5     1 WC           18 DET       NO                7         7 Home        -18
 6     1 WC           18 DAL       NYG               7         7 Home         17
 7     1 DIV          19 PIT       MIA               7         7 Home          2
 8     1 DIV          19 BAL       NYJ               7        14 Home         10
 9     1 DIV          19 ARI       NYG               7         7 Home         10
10     1 DIV          19 DET       CHI               7        14 Home          1
11     1 CON          20 MIA       NYJ               7         7 Home          9
12     1 CON          20 NYG       CHI               7         7 Home          3
13     1 SB           21 CHI       NYJ              14        14 Neutral     -22
```

Coming off the playoff bye, the Bears squeaked out a 1 point victory over the Lions,
won the NFC Conference Championship by 3 points, but then decisively took care of
the Jets in the Super Bowl.

As you may have gathered at this point, the default simulation code picks a random
Elo for every team, and uses those as the starting Elo ratings for all 32 teams.
However, the default code Elo will adjust indepedently within each simulation as
each week is simulated. (The Elo model used is loosely based off of that of
[FiveThirtyEight](https://fivethirtyeight.com/methodology/how-our-nfl-predictions-work/).)

## Use Your Own Model

But of course the real value is putting in your own model into the simulatior. To
accomplish this, you can write your own function which will determine the output of
games instead. As an example, here's a very stupid model that makes the team earlier
alphabetically win by 3 points 90% of the time, and lose by 3 points the other 10% of
the time.

``` r
stupid_games_model <- function(t, g, w, ...) {
  # make the earlier alphabetical team win 90% of the time
  g <- g %>%
    mutate(
      result = case_when(
        week != w ~ result,
        away_team < home_team ~ sample(c(-3, 3), n(), prob = c(0.9, 0.1), replace = TRUE),
        away_team > home_team ~ sample(c(-3, 3), n(), prob = c(0.1, 0.9), replace = TRUE),
        TRUE ~ 0
      )
    )
  
  # return values
  return(list(teams = t, games = g))
}
```

When you create this function, the first two inputs are data on the teams (one row
per team per sim), and data on the games (one row per game per sim). The third argument
is the week number currently being simulated, as only one week is processed at a time.

Your function's job - by whatever means you choose - is to update the `result` column for
that week's games in each of the sims with the number of points the home team won by
(or lost by if negative, or 0 if the game ended in a tie).

It returns both the `teams` and the `games` data. It does both because this way you can
store information in new columns by team or by game to use in the next call. Make sure
your code both accepts and returns the appropriate information, or the simulator will
break!

For example, the default function updates a team's Elo after the game, and stores it
in the `teams` data. When the simulator processes the next week, it uses the updated Elo
rating to inform the team's next game.

!! Also, make sure you aren't overriding completed games or games that aren't in the current
week of `w`. The simulator will **not** stop you from setting past, present, or future
game results in your function, whether you meant to do so or not. !!

Let's run a simulation with `stupid_games_model` and see what happens:

``` r
> sims2 <- simulate_nfl(process_games = stupid_games_model, fresh_season = TRUE)
* 2021-02-13 22:19:10: Loading games data
* 2021-02-13 22:19:11: Beginning simulation of 1000 seasons in 1 rounds
* 2021-02-13 22:20:30: Combining simulation data
* 2021-02-13 22:20:30: Aggregating across simulations
> sims2$overall %>% arrange(team) %>% head()
# A tibble: 6 x 11
  conf  division  team   wins playoff  div1 seed1 won_conf won_sb draft1 draft5
  <chr> <chr>     <chr> <dbl>   <dbl> <dbl> <dbl>    <dbl>  <dbl>  <dbl>  <dbl>
1 NFC   NFC West  ARI    14.4   0.997 0.982 0.423    0.757  0.676      0      0
2 NFC   NFC South ATL    14.4   1     0.899 0.509    0.188  0.169      0      0
3 AFC   AFC North BAL    14.4   1     0.843 0.596    0.78   0.118      0      0
4 AFC   AFC East  BUF    13.5   1     1     0.278    0.162  0.02       0      0
5 NFC   NFC South CAR    12.1   0.969 0.101 0.024    0.036  0.007      0      0
6 NFC   NFC North CHI    12.7   0.988 0.896 0.039    0.014  0.002      0      0
> sims2$overall %>% arrange(team) %>% tail()
# A tibble: 6 x 11
  conf  division  team   wins playoff  div1 seed1 won_conf won_sb draft1 draft5
  <chr> <chr>     <chr> <dbl>   <dbl> <dbl> <dbl>    <dbl>  <dbl>  <dbl>  <dbl>
1 AFC   AFC North PIT    3.24       0     0     0        0      0  0.007  0.351
2 NFC   NFC West  SEA    3.89       0     0     0        0      0  0.012  0.382
3 NFC   NFC West  SF     2.42       0     0     0        0      0  0.198  0.845
4 NFC   NFC South TB     1.60       0     0     0        0      0  0.221  0.832
5 AFC   AFC South TEN    1.66       0     0     0        0      0  0.184  0.848
6 NFC   NFC East  WAS    1.59       0     0     0        0      0  0.344  0.888
```

As you might expect, the earliest alphbetical teams win a lot. The Cardinals
win the Super Bowl in 68% of seasons! Meanwhile, the teams at the bottom
alphabetically are virtually certain to be at the top of the draft order.

## Adding In Your Own Data

This is all well and good, you might be thinking, but your model works off of
other data not in the simulator! How can that work. This is where we utilize R's
ability to have generic arguments.

The `...` at the end of the function definition means that the function can be
called with any number of additional arguments. You can name these whatever you
want, as long as they're not already the name of other defined arguments.

When you call the `simulate_nfl()` function, it too uses the `...` syntax, which
allows you to pass in any number of additional arguments to the function. The
simulator will in turn pass these on to *your* function that processes games.

For example, let's slightly modify our last example:

``` r
biased_games_model <- function(t, g, w, ...) {
  
  # arguments
  args <- list(...)
  best <- ""
  worst <- ""
  
  # best team?
  if ("best" %in% names(args)) {
    best <- args$best
  }
  
  # worst team?
  if ("worst" %in% names(args)) {
    worst <- args$worst
  }

  # make the best team always win and the worst team always lose
  # otherwise, make the earlier alphabetical team win 90% of the time
  g <- g %>%
    mutate(
      result = case_when(
        week != w ~ result,
        away_team == best | home_team == worst ~ -3,
        away_team == worst | home_team == best ~ 3,
        away_team < home_team ~ sample(c(-3, 3), n(), prob = c(0.9, 0.1), replace = TRUE),
        away_team > home_team ~ sample(c(-3, 3), n(), prob = c(0.1, 0.9), replace = TRUE),
        TRUE ~ 0
      )
    )
  
  # return values
  return(list(teams = t, games = g))
}
```

This allows us to define `best` and `worst`, and use that information to
determine a result. While `best` and `worst` are in this example single-length
character vectors, they can be data frames or any other R data type.

Let's simulate using this:

``` r
> sims3 <- simulate_nfl(process_games = biased_games_model, fresh_season = TRUE, best = "CHI", worst = "GB")
* 2021-02-13 22:48:58: Loading games data
* 2021-02-13 22:48:58: Beginning simulation of 1000 seasons in 1 rounds
* 2021-02-13 22:50:15: Combining simulation data
* 2021-02-13 22:50:16: Aggregating across simulations
> sims3$overall %>% filter(team == "CHI")
# A tibble: 1 x 11
  conf  division  team   wins playoff  div1 seed1 won_conf won_sb draft1 draft5
  <chr> <chr>     <chr> <dbl>   <dbl> <dbl> <dbl>    <dbl>  <dbl>  <dbl>  <dbl>
1 NFC   NFC North CHI      16       1     1 0.967        1      1      0      0
> sims3$overall %>% filter(team == "GB")
# A tibble: 1 x 11
  conf  division  team   wins playoff  div1 seed1 won_conf won_sb draft1 draft5
  <chr> <chr>     <chr> <dbl>   <dbl> <dbl> <dbl>    <dbl>  <dbl>  <dbl>  <dbl>
1 NFC   NFC North GB        0       0     0     0        0      0  0.995      1
```

And this shows exactly what we expect. By defining the Bears as the best team,
they always go 16-0, win the division, and win the Super Bowl. Interestingly,
they do not always get the #1 seed. This however makes sense. Since in games 
without the Bears or the Packers the alphabetically earlier teams still wins 
90% of the time, the Cardinals would be expected to go 16-0 in some simulations,
and have tiebreakers over the Bears. However, they'll still ose to them in the end.

Similarly, the Packers always go 0-16, and never have any kind of realistic
postseason. They do almost always get the #1 draft pick -- but not always. Using the same
logic as above, sometimes the Washington Football Team will go 0-16 too, and may
beat the Packers out for the #1 pick through tiebreakers.

## Simulation Configuration

There is a lot of flexibility in how you choose to run the simulation. These are
the parameters and how to configure them when you run the `simulate_nfl()` function.

* **nfl_season** - Which NFL season are you simulating? By default, it simulates
the most recent season for which the regular season schedule is available through
[Lee Sharpe's NFL game data](https://github.com/leesharpe/nfldata/tree/master/data).
This data only goes back to the 1999 season.
* **process_games** - This is where you supply a function you've written to encompass
your model used to determine simulated games results, like the examples above. By
default, this will generate a random Elo for every team per round of simulations, then
use that to determine game data.
* **playoff_seeds** - How many playoff seeds per conference are used? By default, this
is 7 for seasons 2020 and later, and 6 for earlier seasons.
* **if_ended_today** - This should only be used when running in the middle of the
regular season. It will take all completed games as done, but remove the rest of the
regular season games from the schedule, and begin the playoffs as if everything was
locked in based on the regular season data that exists so far.
* **fresh_season** - You'll see this was set to `TRUE` in all of our examples above.
This setting deletes any playoff games and clears out the results for all regular 
season games, so everything is generated fresh. The default is `FALSE` where all games
that have been completed in real life are treated as locked in, and instead remaining
games are simulated.
* **fresh_playoffs** - Similar to `fresh_season`, except instead when set to `TRUE`,
regular season results remain and only playoff games are deleted and then simulated.
The default is `FALSE` in which case playoff games that are completed are accepted
as they occurred,
* **tiebreaker_depth** - How far do you want tiebreakers to be utilized? Usually 
leaving it at the default below (`3`) is fine, but before the season starts, you
may wish to have less tiebreaking in order to 
  * `1`: All teams with the same record have any ties broken randomly.
  * `2`: Instead of evaluating common games if that step is reached, break any ties
randomly. But all earlier tiebreakers are handled correctly.
  * `3`: The default. All tiebreakers are handled through strength of schedule are
processed (or through strength of victory for draft pick order). In the unlikely
event of a further tie, it will be broken randomly.
* **simulations** - How many simulations should be run? Defaults to 1000.
* **sims_per_round** - The simulator can break things up into chunks of simualted
seasons, process each chunk on its own (called a round), and then aggregate everything
together at the end. Defaults to `simulations` or 1000, whichever is lower. If your
computer is hanging and forces a restart while running this simulation, it is recommended
that you lower this number.

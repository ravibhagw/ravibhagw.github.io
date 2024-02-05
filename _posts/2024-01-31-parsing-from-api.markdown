---
layout: post
title:  "Pulling EASHL Statistics Programmatically"
date:   2024-01-30 17:28:00 -0500
categories: personal-projects
---

As far as side projects go, I've always come back to trying to pull EA Sports NHL stats from their (undocumented) API for a private league I play in during my spare time.  

When it comes to side projects, I love building things and exploring new concepts, but it has to have a purpose.  After 12 years of professional experience plus and having started coding as a hobby as a teenager, I don't get much joy from building the todo apps, ecommerce apps, or weather forecast APIs that so many tutorials and pet projects try to guide you though. Instead I like to look for projects that scratch some creative or analytical itch, or otherwise allow me to solve some problem I find personally interesting.  I save the other stuff for work-specific learning. 

I've had blog posts on the topic of this league before, but up until now all of the data I've pulled has been from the league's website rather than the EA Sports stats website. 

I had talked a little bit about it previously in the [Discord bot post](https://ravib.dev/personal-projects/2023/10/14/discord.html) where I mentioned that the code that pulled everything into a SQLite database was being repurposed for the Discord bot.  You can consider this post a deep dive into the original code :)

## Pulling Data

The data comes from EA Sports' APIs that power their EASHL Seasons website.  At one point years ago they exposed OpenAPI docs for these APIs, but those have long-since been removed.  I've always been hesitant to go too deep into building "official" things against this for two reasons:

1. Since they are internal, undocumented, and designed as APIs to power specific frontends, they often cause breaking changes without notice for each edition of the game.

2. They currently do not require any authentication or tokens, but since it is not officially provided for developer purposes that can change at any time

The actual code to pull data is nothing special -- quite literally just an API call into a byte array:

```  
jsonBytes, err := GetRecentClubMatches(teamId)
	if err != nil {
		panic(err)
	}

```

## Parsing Data

This is also pretty straightforward mapping code.  The entrypoint to mapping looks like this:

```
	for _, game := range matchResults {
		var parsedGame models.ParsedMatchView
		parsedGame.MatchId = game.MatchId
		parsedGame.Timestamp = game.Timestamp

		matchDto := ParseMatchDetail(game.MatchId, game.Timestamp, &game.Clubs)
		playerDtos := ParsePlayerDetails(game.MatchId, &game.Players)

		parsedGame.MatchData = matchDto
		parsedGame.Players = playerDtos
		retVal = append(retVal, parsedGame)
	}

```

We're building a viewmodel of the matchup, consisting of stats for each team via `ParseMatchDetail` and each player via `ParsePlayerDetails` with some very basic data for the match itself -- the ID as provided by the API and the timestamp indicating the time the match ended.

Under the hood, those methods take the properties of the object to which we mapped the raw JSON and map it to a more structured and useful object:

```
// Our loosely structured struct with map[string] galore! 
type MatchResultMap struct {
	MatchId   string
	Timestamp int64
	Clubs     map[string]TeamGameStat
	Players   map[string]map[string]SkaterStat
	Aggregate map[string]interface{}
}

type SkaterStat struct {
	Goals             int `json:"skgoals,string"`
	Assists           int `json:"skassists,string"`
	Points            int
	Hits              int     `json:"skhits,string"`
	Plusminus         int     `json:"skplusmin,string"`
	ShotAttempts      int     `json:"skshotattempts,string"`
	Shots             int     `json:"shots,string"`
	ShootingPct       float32 `json:"shokshotpct,string"`
	Deflections       int     `json:"skdeflections,string"`
	Ppg               int     `json:"skppg,string"`
	Shg               int     `json:"skshg,string"`
	Passattempts      int     `json:"skpassattempts,string"`
	Passcomplete      int     `json:"skpasses,string"`
	Passpct           float32 `json:"skpasspct,string"`
	Saucerpasses      int     `json:"sksaucerpasses,string"`
	Offrating         float32 `json:"ratingOffense,string"`
	Defrating         float32 `json:"ratingDefense,string"`
	Teamrating        float32 `json:"ratingTeamplay,string"`
	Blockedshots      int     `json:"skbs,string"`
	Takeaways         int     `json:"sktakeaways,string"`
	Interceptions     int     `json:"skinterceptions,string"`
	Giveaways         int     `json:"skgiveaways,string"`
	PimsDrawn         int     `json:"skpenaltiesdrawn,string"`
	Pim               int     `json:"skpim,string"`
	Pkclears          int     `json:"skpkclearzone,string"`
	PossessionSeconds int     `json:"skpossession,string"`
	Faceoffwins       int     `json:"skfow,string"`
	Faceoffloss       int     `json:"skfol,string"`
	Faceoffpct        float32 `json:"skfopct,string"`
	ToiSeconds        int     `json:"toiseconds,string"`
	Scoredgwg         int     `json:"skgwg,string"`

	Gamertag    string            `json:"playername"`
	ClassPlayed enums.PlayerClass `json:"class,string"`

	PositionSelected string `json:"position"`

	Savepct          float32 `json:"glsavepct,string"`
	Shotsagainst     int     `json:"glshots,string"`
	Saves            int     `json:"glsaves,string"`
	Gaa              float32 `json:"glgaa,string"`
	Breakawayshots   int     `json:"glbrkshots,string"`
	Breakawaysaves   int     `json:"glbrksaves,string"`
	Breakawaysavepct float32 `json:"glbrksavepct,string"`
	Desperationsaves int     `json:"gldsaves,string"`
	Pokechecks       int     `json:"glpokechecks,string"`
	Penaltyshots     int     `json:"glpenshots,string"`
	Penaltyshotsaves int     `json:"glpensaves,string"`
	Penaltysavepct   float32 `json:"glpensavepct,string"`
	ShutoutPeriods   int     `json:"glsoperiods,string"`

	TeamId   string
	TeamName string
}

// Our better-structured documents

type SkaterStatDTO struct {
	Player_ea_id string
	Player_class int
	Match_id     string
	Gamertag     string
	Position     string

	Goals              int
	Assists            int
	Points             int
	Hits               int
	Plusminus          int
	Shots              int
	ShotAttempts       int
	Shootingpct        float32
	Deflections        int
	Ppg                int
	Shg                int
	Pass_attempts      int
	Pass_complete      int
	Pass_pct           float32
	Pass_sauces        int
	Off_rating         float32
	Def_rating         float32
	Team_rating        float32
	Blocked_shots      int
	Takeaways          int
	Interceptions      int
	Giveaways          int
	Penalties_drawn    int
	Pims               int
	Penalties_clears   int
	Possession_seconds int
	Faceoff_wins       int
	Faceoff_loss       int
	Faceoff_pct        float32
	Scored_gwg         int

	Goalie_save_pct          float32
	Goalie_shots_against     int
	Goalie_saves             int
	Goalie_gaa               float32
	Goalie_breakaway_shots   int
	Goalie_breakaway_saves   int
	Goalie_Breakawaysavepct  float32
	Goalie_desperation_saves int
	Goalie_pokechecks        int
	Goalie_penaltyshot       int
	Goalie_penaltyshot_saves int
	Goalie_penalty_savepct   float32
	Goalie_shutout_periods   int
	TeamId                   int
}

```


## Match Analysis Complications

The API call itself is fairly simple.  You can filter by match types.  Thankfully they provided a querystring to specify private matches, which is what we use to operate our league games
```
https://proclubs.ea.com/api/nhl/clubs/matches?matchType=club_private&platform=common-gen5&clubIds=[id]
```

The complications arise when you try to separate league matches from non-league matches.  Sometimes people like to:

* Play private games before game start times as warmups/scrimmages
* Play private games on off-nights against other teams 
* Play private games on nights that are off-nights for some teams but not other teams


To solve this we have to use a set of heuristics and execution rules.  

First and foremost, the job is scheduled to run in a way that won't capture games played on Friday or Saturday, as those are off-nights that nobody plays league games in.  We address this via scheduling rather than in-code rules because on rare occasions a reschedule may happen on a Friday or Saturday.

Secondly, we use some code rules to ensure that we only capture:

1. Matches between two teams in the league
2. Matches that end at reasonable end times for league games

```
func isXbsClubs(match dal.MatchStatDTO) bool {

	teamIdMap := map[string]int{
        // Map
	}

	homeTeamIsXbs := false
	awayTeamIsXbs := false
	for _, value := range teamIdMap {
		if value == match.Home_team_id {
			homeTeamIsXbs = true
		}
	}

	for _, value := range teamIdMap {
		if value == match.Away_team_id {
			awayTeamIsXbs = true
		}
	}
	return (homeTeamIsXbs && awayTeamIsXbs)
}

func isTimeInRange(epochTimestamp int64) bool {
	t := time.Unix(epochTimestamp, 0)

	location, err := time.LoadLocation("America/New_York")
	if err != nil {
		fmt.Println("Error loading location:", err)
		return false
	}
	t = t.In(location)

	// Define start and end times (10:15 PM to 11:40 PM)
	startTime := time.Date(t.Year(), t.Month(), t.Day(), 22, 15, 0, 0, location)
	endTime := time.Date(t.Year(), t.Month(), t.Day(), 23, 40, 0, 0, location)

	return t.After(startTime) && t.Before(endTime)
}

```


The third item to implement is still in-progress, but it is essentially a schedule-checker.  Any time we load data, we should additionally check the league website to ensure that there's a scheduled match between these two teams.  That will help eliminate cases where, say, San Jose and Colorado are not scheduled to play but decide to have a fun game anyways.  The blocker here is that all of that code lives in my .NET projects and isn't written in a way that it can be separated into a service that the Golang code can consume.  

The fourth challenge is a simple API limitation in that it only returns the last 5 matches for any match type.  That makes timing important, because there's no way to capture the raw data once it's gone.  My guess is that EA captures the data for a period of time, runs some job to summate all-time stats for the players, then purges data.  

The last challenge, and one I haven't quite solved yet, is how to handle disconnects.  Occasionally networks do network things and players disconnect.  Our rules state that players disconnecting in the first period should be allowed back in for period 2, meaning we have to quit the game and restart.  Goalies disconnecting force-quit the games, and thus force a reset.  Sometimes if the player or goalie cannot come back (ISP outage, emergency, power issue) we need to bring in a different player/goalie.  Sometimes this results in a complete line shuffling.

This results in multiple problems:

1. Partial games need to be summed up to one match, where 1...n results from the API correspond to a single match played for the league
2. Total number of players and player positions may not align between partial matches
3. End times are unpredictable.  A first period lag-out wouldn't be caught by our 10:15 heuristic
4. A lag-out in overtime would be difficult to detect as a partial match
4. Teams may abandon a no-event first period lagout and restart from scratch, abandoning that partial match

I've not had time to chew on these problems yet, but I'd imagine there's some way to solve them, even if manual intervention is required.  


## Calculating Advanced Stats

With that data parsed and stored in a table with the following definition, we can start having fun with advanced stats:

```
CREATE TABLE "SkaterStats" (
    Player_ea_id             TEXT,
    Player_class             INTEGER,
    Match_id                 TEXT,
    Gamertag                 TEXT,
    Position                 TEXT,
    Goals                    INTEGER,
    Assists                  INTEGER,
    Points                   INTEGER,
    Hits                     INTEGER,
    Plusminus                INTEGER,
    Shots                    INTEGER,
    ShotAttempts             INTEGER,
    Shootingpct              REAL,
    Deflections              INTEGER,
    Ppg                      INTEGER,
    Shg                      INTEGER,
    Pass_attempts            INTEGER,
    Pass_complete            INTEGER,
    Pass_pct                 REAL,
    Pass_sauces              INTEGER,
    Off_rating               REAL,
    Def_rating               REAL,
    Team_rating              REAL,
    Blocked_shots            INTEGER,
    Takeaways                INTEGER,
    Interceptions            INTEGER,
    Giveaways                INTEGER,
    Penalties_drawn          INTEGER,
    Pims                     INTEGER,
    Penalties_clears         INTEGER,
    Possession_seconds       INTEGER,
    Faceoff_wins             INTEGER,
    Faceoff_loss             INTEGER,
    Faceoff_pct              REAL,
    Scored_gwg               INTEGER,
    Goalie_save_pct          REAL,
    Goalie_shots_against     INTEGER,
    Goalie_saves             INTEGER,
    Goalie_gaa               REAL,
    Goalie_breakaway_shots   INTEGER,
    Goalie_breakaway_saves   INTEGER,
    Goalie_Breakawaysavepct  REAL,
    Goalie_desperation_saves INTEGER,
    Goalie_pokechecks        INTEGER,
    Goalie_penaltyshot       INTEGER,
    Goalie_penaltyshot_saves INTEGER,
    Goalie_penalty_savepct   REAL,
    Goalie_shutout_periods   INTEGER,
    TeamId                   INTEGER,
    Is_xbs_league_game       INTEGER DEFAULT (0),
    CONSTRAINT unique_player_match PRIMARY KEY (
        Player_ea_id,
        Match_id
    )
    ON CONFLICT IGNORE
);
```

Some interesting stats I've looked at so far include:

* Pass Attempts / Giveaways - tracking whether a high turnover player is potentially forcing passes, turning over the puck trying to make plays, or coughing up the puck holding it too long

* Possession Time / Giveaways - similar to the above, trying to guestimate how this player plays whether they're losing the puck a lot or not

* Saucer Passes / Passes Completed - Assessing if this player is an "advanced" passer, either via manual saucer passes or auto-saucer passes that the game provides

* SHG / Penalties_Clears - Looking to see how "impactful" this player is as a penalty killer.  Are they a high risk/high reward player on the penalty kill?  Are they a low risk player looking to dump the puck?  This is especially useful for right wingers to understand how they're handling their defensive responsibilites when a defenseman takes a penalty

* Penalties_drawn / Possession Time - Is this player good at controlling and shielding the puck?  Are they forcing the opposing team into penalties to strip them of it?

There's plenty more potential here, including calculting corsi, looking at goalies, looking at players' overall scores and distribution of Off/Def/Team Ratings, etc.  The data is not perfect but with it captured it's very simple to write queries to summate/average data per player and run these calculations. 
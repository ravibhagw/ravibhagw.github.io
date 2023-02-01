---
layout: post
title:  "Creating the XBS Card Maker - Part 3"
date:   2023-01-29 15:20:57 -0500
categories: personal-projects
---

As a tech geek at heart, I love finding ways of using code build fun things that reasonate with me. One of the more interesting personal projects I took on last year was building a hockey card maker for an online hockey game league I like to play in a couple of nights a week.  

This post seems like a great way to break down that project and summarize the approach.  As you'll discover, I ran into some interesting challenges putting this together. All in all this project looked deceptively simple on the surface, but wound up being tremendously complex.  It's one of the more complicated things I've taken on for a side project, and one of the first times I've melded an idea, a bit of creativity, and a lot of code to make something interesting.

## The Idea

Build hockey cards for the end of the season for every player in the league, in the same vein as your average hockey card.

![sample hockey card](/assets/s-l1600.jpg)

After looking through several hockey cards online, I decided on the basic structure of the card:
- Player's name (gamertag)
- Team name
- Position
- Stats for the season
- Description of the player
- Some graphics, that I'd figure out later

## Part 3: Assembling the Cards with Code

With the data I needed and just enough graphic editing prowess, I was finally ready to start building the hockey cards themselves.

The hockey card generation code was also built in C#.  At a high level, it does the following:

1. Reads in the player data from file
2. Reads in the team data from file
3. For each player:
    1. Accumulate all stats for the season across multiple positions 
    2. Determine the most frequently played position so that the correct stats line could be rendered
        1. Remember from part 1 that Centers, Wingers, Defensemen, and Goalies all have stats displayed differently
    3. Determine the last team the player played for
    4. Generate the card, including:
        1. Stats
        2. Gamertag
        3. Team Logo
        4. Xbox Avatar
    5. Save the card

Simple enough, right?

### Accumulating Stats with LINQ and Reflection

The models were designed such that each `Player` object would have a `PlayerAnalytics` object to house statistics.  In addition to having simple data captured directly from the site, it would expose properties with pre-calculated advanced statistics (per-game attributes, averages, min/max, etc).

Using that model, we needed to accumulate stats.  `PlayerAnalytics` intentionally separated stats per-season and per-position.  For a hockey card, we wanted to capture all of the games a player played and their status. Thus, we would build a new object, `AccumulatedStats`, that would perform all of the heavy-lifting we needed for the card.


    public class AccumulatedStats
    {
        private Player _player;

        public string Position { get; }
        public string Gamertag { get;  }
        public string LastTeam { get; }

        public int? GamesPlayed { get; }
        public int? Goals { get; }
        public int? Assists { get; }
        public int? Points { get; }
        public int? PenaltyMinutes { get; }
        public int? PlusMinus { get; }
        public decimal? DRating { get; }
        public int? Hits { get; }
        public int? Shots { get; }
        
        public int? GoalieGamesPlayed { get; }
        public int? GoalieWins { get; }
        public int? GoalieLoss { get; }
        public int? GoalieOTL { get; }
        public int? GoalieGoalsAgainst { get; }
        public int? GoalieSaves { get; }
        public decimal? GoalieGAA { get; }
        public decimal? GoalieSavePct { get; }
        public int? GoalieShutouts { get; }
    }

You'll notice that goalie stats are separated.  Goalies use a completely different set of statistics.  When we render the card later, we look at `Position` to determine how to render the stats line.  

Fortunately, I had designed all of the models in `PlayerAnalytics` for the project using `Collections` in C#.  LINQ is such a powerful feature, and something I sorely miss in other languages. It allows you to query in-memory collections similar to SQL in that you write expressive code, requesting *what* you want without having to specify *how* to retrieve it.

`GoalieGamesPlayed = player.PlayerAnalytics?.GoalieStats.Where(x => x.Season == TARGET_SEASON).Sum(x => x.GamesPlayed);`

Goalies were easy.  They were a position unto themselves, so we just had to use the power of LINQ to grab what we needed.  Search the `GoalieStats` property for the target season and calculate the sum of the requested property.

Skaters were trickier, because we had to accumulate the Center, Wing, and Defense stats. 

    Goals = Accumulate(player, "Goals");
    
    private int? Accumulate(Player player, string propertyName)
    {       
        return player.PlayerAnalytics?.CenterStats.Where(x => x.Season == TARGET_SEASON).Sum(x => GetStatToAccumulate(x, propertyName))
           + player.PlayerAnalytics?.WingerStats.Where(x => x.Season == TARGET_SEASON).Sum(x => GetStatToAccumulate(x, propertyName))
           + player.PlayerAnalytics?.DefenseStats.Where(x => x.Season == TARGET_SEASON).Sum(x => GetStatToAccumulate(x, propertyName));
    }

    private int GetStatToAccumulate<T>(T stats, string propertyName)
        where T: Core.SkaterStatistics
    {
        return (int)(stats.GetType().GetProperty(propertyName).GetValue(stats) ?? 0);
    }

Here we get to benefit from the power of reflection, LINQ, and the ability to use weak-typing in a strongly-typed language, summing up the values from each position for the target season with a weakly-typed property field, allowing us to re-use the `Accumulate` method across any integer-based stat (I.E most of them).


Reflection allowed me to use weak typing.  I could specify a field name and use reflection alongside LINQ to accumulate the stats where needed.  

### Retrieving the Xbox Avatar with a GET request

Microsoft was nice enough to leave their legacy avatars from the Xbox 360 eras available.  I wouldn't be surprised if there's some legacy services that still dependent on it. 

You can request them from: `https://avatar-ssl.xboxlive.com/avatar/YOUR_GAMERTAG/avatar-body.png`

Mine looks like this:

![avatar](/assets/avatar-body.png)

I also added a generic avatar, in case the request fails or the gamertag could not be looked up.  The code assumes that the user's gamertag is the same as the user's username on our stat tracking site, which is not always true.  Unfortunately this API is not compatible with the new hashtag suffixes.  At this point it's safe to assume that these legacy avatars, deprecated around 2013, are used by any user with a hashtag suffix gamertag, which was launched in 2019.

![avatar2](/assets/avatar-body-generic.png)
### Resolving the Team

Choosing the right team and right position for the card proved to be challenging. The team chosen should be the last team the player played on, and the position should be the position with the most games played. 

Players can play for more than one team during a season if they are traded.  The data scraper attempts to pull the stats across two dimensions: 
1. Position
2. Season

To add a third dimension of Team would have complicated the screenscraping to an extent I didn't want to deal with for a side project. 

There's good news and bad news around that.  The good news is that the site exposes the team names the player played for *and* lists them in order, which we capture.  The bad news is that this isn't captured globally, but on a per-position basis.

This makes resolving the right team and the right position challenging when players get traded but play different positions across different teams.  Consider the following:

| Position | Team Played |
| --- | --- |
| Center | Team 1, Team 3 |
| Defense | Team 2, Team 3 |
| Wing | Team 1  |

Here we have a player who played wing on Team 1, defense on Team 2, and a mix of center and defense on Team 3.

How do you choose the right team, when the teams played list is per position? Since it's an ordered list per position, it was simple to merge the list into a hashmap of its positions, and resolve the positions:

| Team | Indicies |
|---|---|
|Team 1| 0, null, 0 |
|Team 2| 0, null, null |
|Team 3| 1, 1, null |

From there I had some logic to determine what the most appropriate index was.  The logic was not that robust and guessed the start/middle indexes, but was accurate in all cases for the last team, which was the only one we cared about. 

### Resolving the Position

Now that we have the player's team, we need to select the best appropriate position for the team.  This was another neat little piece of LINQ code, thanks to enumeration and collections in C#.

First, create a hashmap of the positions and the number of games played at that position:

`Dictionary<string, int?> valueMap = new Dictionary<string, int?>();`

Once we'd inserted the key/value pairs for each position in, we can run this piece of code:

`var position = valueMap.Aggregate((selected, next) => selected.Value > next.Value ? selected : next).Key;`

`Aggregate()` iterates over every element in the collection, and foreach element evaluates the supplied function.  The returned value becomes the `selected` value passed to compare against the `next` value, solving for the max value and returning the `Key` (position name).

### Rendering the Card

Finally, with all of the data elements ready we could build the card.

    public Bitmap CreateCard(AccumulatedStats player, string template, bool includeGamerpic = false)
    {
        Bitmap imageCache = new Bitmap(template);
        using (Graphics g = Graphics.FromImage(imageCache))
        {
            AddAvatar(g, player);
            AddGamertag(g, player);
            AddStats(g, player);
            AddLogo(g, player);
            AddPrimaryPosition(g, player);
            AddSynopsisArea(g);
        }
        return imageCache;
    }

The `AccumulatedStats` object had all of the data points, the image template could be read in from a provided path, and all of the pieces could be published correctly.

The `CreateCard()` method could be involved to take in the stats object and template information, load the template, assemble the image, and save to disk:

`cardMaker.CreateCard(stats, $"C:\\backgrounds\\{(String.IsNullOrEmpty(teamName) ? "generic_hut" : teamName)}.png").Save($"D:\\test\\{teamName}\\{player.Gamertag}.png", System.Drawing.Imaging.ImageFormat.Png);`

The only particularly interesting pieces of code here are:

1. AddAvatar, which makes the GET request we discussed above
2. Dynamic sizing for the Gamertag, as seen below:

        private void AddGamertag(Graphics g, AccumulatedStats player)
        {
            // Dynamically resize the font to fit the area.
            var fontSize = 38;
            var bebas = new Font("Bebas", fontSize, FontStyle.Bold | FontStyle.Italic, GraphicsUnit.Point);
            while (g.MeasureString(player.Gamertag, bebas).Width > 334)
            {
                fontSize -= 2;
                bebas = new Font("Bebas", fontSize, FontStyle.Bold | FontStyle.Italic, GraphicsUnit.Point);
            }
            
            var rightAlign = new StringFormat() { Alignment = StringAlignment.Far };
            g.TextRenderingHint = System.Drawing.Text.TextRenderingHint.AntiAliasGridFit;
            g.DrawString(player.Gamertag, bebas, Brushes.Black, new PointF(470, 410), rightAlign);
            
        }

With a fixed width for area to support the gamertag and variable gamertag lengths, I had to loop over the width of the rendered string and decrement by two font size untils until the string would fit the desired canvas area.

### The Result

The code worked well.  With the supplied input data it was able to iterate over every player and build a card for each of them.

![](/assets/final_card.png)

The graphics and design certainly leave something to be desired (remember, I'm not a graphics person!), but the key elements are all here:

- Player's name (gamertag)
- Team name
- Position
- Stats for the season
- Description of the player
- Some graphics


The summary needed to be written manually, which I did after quickly glancing over the player's stats. 

All things considered though, the investment in this code saved a tremendous amount of time.  I was able to generate 160 cards across 16 teams relatively effortlessly.  The full set of cards are here:

- https://imgur.com/a/oxBAurj
- https://imgur.com/a/rVyIug9
- https://imgur.com/a/pkRzFTV
- https://imgur.com/a/38I5ohz
- https://imgur.com/a/Q9uA5Cr
- https://imgur.com/a/XVKURYv
- https://imgur.com/a/Dhn3pgk
- https://imgur.com/a/iNm6qqw
- https://imgur.com/a/iEKeubY
- https://imgur.com/a/Wa0nnoA
- https://imgur.com/a/nhViY8f
- https://imgur.com/a/LeT61pq
- https://imgur.com/a/0B1GVH1
- https://imgur.com/a/nvJd3QZ
- https://imgur.com/a/BA8fYLM
- https://imgur.com/a/roRqSzc
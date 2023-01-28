---
layout: post
title:  "Creating the XBS Card Maker"
date:   2023-01-29 15:20:55 -0500
categories: personal-projects
---

As a tech geek at heart, I love finding ways of using code to do interesting things. One of the more interesting personal projects I took on last year was building a hockey card maker for an online hockey game league I like to play in a couple of nights a week.  

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

## Getting the Data

Retrieving the data would be the first problem.  The league has 100+ players across 10+ teams, and doing all of this by hand would not be feasible in the slightest.  

Fortunately I was able to re-use some code I had put together for a different side-project to screenscrape the site.  That code is written in C#, based on the [HtmlAgilityPack](https://html-agility-pack.net/), and available on my [GitHub](https://github.com/ravibhagw/XBS.Core.Parser/blob/master/Parser.cs)

I pulled all of that into a [data scraper project](https://github.com/ravibhagw/XBSDataScraper/blob/master/Program.cs) that would:

1. Scrape player data for the specific season
2. Scrape team data for the specific season
3. Persist the data to file

        var parser = new Parser(
            targetSeason: "30",
            seasonStatsToPull: new int[] { 30 }
            );

        var players = parser.GetPlayers(); // Method from Parser 
        var statsTeams = parser.GetTeams(); // Method from Parser

        using (FileStream file = new FileStream("teams_stats.xml", FileMode.Create, FileAccess.Write))
        {
            System.Xml.Serialization.XmlSerializer ser = new System.Xml.Serialization.XmlSerializer(typeof(List<StatsTeam>));
            ser.Serialize(file, statsTeams);
        }

        using (FileStream file = new FileStream("c:\\testoutputdraft\\players_staging.xml", FileMode.Create, FileAccess.Write))
        {
            System.Xml.Serialization.XmlSerializer ser = new System.XmlSerialization.XmlSerializer(typeof(List<Player>));
            ser.Serialize(file, players);
        }

The resulting files looked something like this:

### Teams
    <StatsTeam>
        <Season>30</Season>
        <TeamName>Boston Bruins</TeamName>
        <TeamAbbreviation>BOS</TeamAbbreviation>
        <!-- Others stats, not used for this app -->
    </StatsTeam>

The key piece here is the `TeamAbbreviation`, which is used as a key to link to each player. 

### Players

    <Player>
        <Gamertag>Player_Gamertarg</Gamertag>
        <!-- Other Stats omitted --> 
        <CenterStats />
        <DefenseStats>
            <DefenseStatistics>
                <GamesPlayed>3</GamesPlayed>
                <Season>30</Season>
                <!-- Player played 3 games at defense across 2 teams this season -->
                <TeamNames>
                    <string>Anaheim</string>
                    <string>Minnesota</string>
                </TeamNames>
                <!-- Stats omitted for readability --> 
            </DefenseStatistics>
        </DefenseStats>
        <GoalieStats />
        <WingerStats>
            <WingerStatistics>
            <GamesPlayed>17</GamesPlayed>
            <Season>30</Season>
            <TeamNames>
                <string>Anaheim</string>
                <string>Minnesota</string>
            </TeamNames>
            <!-- Stats omitted for readability -- >
            </WingerStatistics>
        </WingerStats>
        </PlayerAnalytics>
    </Player>

You'll already notice some complexity here, which will rear its head later.  

Namely:
- Player can have played multiple positions, with each position having different categories of statistics
  - Centers have Faceoff stats, but Winger and Defensemen do not
  - Defensemen have D-Rating stats, but Centers and Wingers do not
  - Goalies have a completely separate set of stats
- Players can have played for multiple teams 

Regardless, I now had the data I needed for the cards to be built.  The next thing I needed was the cards themselves. 

## Designing the Card 

This is where I ran into my first major problem - I am not a graphics designer! I'm admittedly terrible at anything graphics related.  It's why I spent most of my software career as a backend developer.  Given a layout, sure, I can mimic what you want.  Ask me to design something myself and you'll get the classic "second year CS grad" output.

## Assembling the Card


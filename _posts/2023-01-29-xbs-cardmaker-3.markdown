---
layout: post
title:  "Creating the XBS Card Maker - Part 3"
date:   2023-01-29 15:20:57 -0500
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

Fortunately, I had designed all of the models for the project using `Collections` in C#.  LINQ is such an amazingly powerful feature, and something I sorely miss in other languages. It allows you to query in-memory collections similar to SQL in that you write expressive code, requesting *what* you want without having to specify *how* to retrieve it.

TODO: Request target season

Reflection allowed me to use weak typing.  I could specify a field name and use reflection alongside LINQ to accumulate the stats where needed

### Retrieving the Xbox Avatar with a GET request

// Retrieving the card via xbox

### Last Team Played

// Pretty easy

### Resolving the Team Names

Team names are weird.  If you played on too many teams in one season, the site would abbreviate them.

Side note, I love OrdinalIgnoreCase.


End result was this:

120 cards generated in under a minute.  
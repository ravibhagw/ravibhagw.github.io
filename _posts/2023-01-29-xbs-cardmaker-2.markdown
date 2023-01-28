---
layout: post
title:  "Creating the XBS Card Maker - Part 2"
date:   2023-01-29 15:20:56 -0500
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

## Part 2: Designing the Card 

This is where I ran into my first major problem - I am not a graphics designer! 

I'm terrible at anything graphics related.  It's why I spent most of my software career as a backend developer.  Given a layout, sure, I can mimic what you want.  Ask me to design something myself and you'll get the classic "second year CS grad" output.

Fortunately I had a bit of experience with Paint.NET. I had used it for modifying screenshots, making the occasional meme, or light photo enhancement, and figured it would be suitable enough for now. 

After playing around with a few different formats, I landed on a template.

First, some templates I didn't go with:
![first work in progress, abandoned](/assets/stats_project-alt.png)
![second work in progress, later reused](/assets/vegas_censored.png)
Note: I later re-used this Vegas template for some draft day cards.  The original version of this was not nearly as well-polished

The final version looked something like this:
![final](/assets/boston_censored.png)

The final choice was simpler, but useful for a few reasons:

1. It required a lot less graphic design skill to create something passable
2. It would let me use each player's Xbox avatar on the card with all of the extra space
3. I could make team-specific templates quickly by editing the border to match team colours

![all cards](/assets/all_cards.png)


The code to build the card could load these templates and use fixed points in the image to render any overlay images or text I needed.  There was lots of blank space and flexibility.  These templates would be an excellent canvas to start assembling the hockey cards. 



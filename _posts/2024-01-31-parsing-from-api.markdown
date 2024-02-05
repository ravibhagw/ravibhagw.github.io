---
layout: post
title:  "Advanced EASHL Stats"
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




## Parsing Data

## Storage

## Calculating Advanced Stats

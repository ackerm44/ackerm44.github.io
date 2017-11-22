---
layout: post
title:      "Creating a Small CLI Data Gem "
date:       2017-11-22 16:11:31 -0500
permalink:  creating_a_small_cli_data_gem
---

You can be assured that upon getting my project to perform as I hoped it would that I did a 2-minute celebration dance.  I should start recording these dances as I’m sure they’re entertainingly nerdy.  

A hobby of mine is gardening, both vegetable and perennial, so in order to pretend that summer hasn’t actually left us for many months, I decided to create a small application that scrapes data from a vegetable gardening website and allows a user to interact with a few menus to select a vegetable to learn more information on growing that vegetable.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Here are a few screenshots of the program:

![Welcome Menu](https://i.imgur.com/fBr1Gg3.png)

![Example of Vegetable Ouput](https://i.imgur.com/p1A5WGc.png)

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

One of the toughest challenges for me when starting a new project is deciding where to even begin.  I wasn't exactly sure how I wanted the output to look but since I have found scraping to be fun and interesting, I figured I would start there. 

Once I felt fairly confident I could scrape what I needed, I had to figure out how to utilize that information in the rest of the program.  I looked at many example projects, followed along in several office hours and watched many prerecorded lectures to find a pattern that would best fit what I was trying to accomplish. 

The biggest trouble I had was in regard to scraping the individual urls for each vegetable. I continually had only the first available url returned rather than all the vegetable urls so after trying many different combinations of selectors and not knowing what I was doing wrong, I decided to use a map method that iterated and collected all urls.  From there, my savior was a simple zip method that allowed me to combine the vegetable url with the name of the vegetable, making initialization of each vegetable so simple!

I have learned so much through this process of not only working through a project from an idea in my head to a finished, executable product, but also of really beginning to understand the importance of good design patterns for efficiency, readability and having the product be less error prone.  These patterns also help me think in terms of overall layout of a project that offer little stepping stones on my way through its design.  This was also my first project of using git and Github directly and after some initial fear of screwing up my program (of which I actually did a few times), I feel a bit more confident with it.  

If you think you might be a gardener at heart, feel free to take this gem for a test drive by visiting the [Github repo here.](https://github.com/ackerm44/cli-data-gem-assessment-v-000/tree/master/gardening-cli-app)


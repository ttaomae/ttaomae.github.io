---
layout: page
title: Robocode
display-order: 0
---
Some details about my experiences with [Robocode](http://robocode.sourceforge.net/) can be found [here]({{ site.baseurl }}{% link _my_tags/robocode.md %}). I will use this page to briefly document my attempts at developing competitive Robocode robots.

# RedShift
[<img src="{{ site.baseurl }}{% link projects/redshift.png %}" alt="robocode redshift" />]({{ site.baseurl }}{% link projects/redshift.png %})

This is my first attempt at a competitive robot. It is primarily a 1-vs-1 robot, but it does make some small attempts to accommodate for melee battles. Its basic strategy is to constantly try to circle the enemy. This constant movement will hopefully make it harder to hit. It uses linear targeting which predicts where the enemy will be by assuming that it will continue to move in the same direction with the same velocity.

More details about this project can be found at this project's [GitHub Page](http://ttaomae.github.com/robocode-tkt-redshift/). There you will find more details about the robot's strategy along with user and developer guides for anyone who wishes to use or develop this robot.

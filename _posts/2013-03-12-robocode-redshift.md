---
layout: post
title: Robocode - RedShift
categories: software-engineering
tags: ics-691 java robocode
---
In my last post I talked about testing the [Robocode](http://robocode.sourceforge.net/) robot which I have been developing for the past few weeks. Today I will be going into more depth about the robot and the project in general.

## Introduction
Although the end product for this project would be a (hopefully) competitive robot, the goals for the project go far beyond simply building a working system. The primary, overarching goal for this project was to learn about and apply various software engineering practices. Among these were [quality assurance]({{ site.baseurl }}{% post_url 2013-02-22-make-it-break %}) and [software testing]({{ site.baseurl }}{% post_url 2013-02-26-robot-testing %}) which I have talked about in previous posts. Other important topics that I learned about throughout this project include coding standards ([Elements of Java Style](http://www.amazon.com/Elements-Java-Style-Reference-Library/dp/0521777682)), automated build systems ([Maven](http://maven.apache.org/)), and version control ([Git](http://git-scm.com/)).

## RedShift
Below I will describe the current state of my robot, RedShift. For the most up-to-date and detailed description, visit this project's [homepage](http://ttaomae.github.com/robocode-tkt-redshift/).

[<img src="robocode_redshift.png" alt="robocode redshift" />](robocode_redshift.png)

### Movement
The basic movement strategy for my robot is to constantly move in a circle around the target while trying to stay between the two blue circles (shown in the image above) which are a predefined distance from the enemy. This constant movement works well against basic targeting strategies. However, for more advanced targeting, this movement is very predictable. To combat this, if the enemy has a high accuracy against my normal movement, I change directions every time the enemy fires. I change between these two strategies depending on how well the enemy is doing against each. Against very advanced targeting strategies, this movement is probably still predictable, but for my purposes it seems to work well.

### Targeting
For targeting I use a well known strategy called [linear targeting](http://robowiki.net/wiki/Linear_Targeting). With linear targeting, each time you scan a robot, you assume that it will continue moving in the same direction with the same speed and you aim where it would be if this assumption were true. I apply a small modification to this strategy to account for the fact that many robots will not continue moving in the same direction or at the same speed. Instead of using the velocity at the time of the scan, I use an average velocity from a number of previous scans. This number changes between rounds while RedShift tries to find the best strategy against the current enemy. To calculate the average, I use a [weighted moving average](http://en.wikipedia.org/wiki/Moving_average# Weighted_moving_average), which gives more weight to recent values.

### Firing
My firing strategy is fairly simple. RedShift will fire at every opportunity as long as it is not farther than a predefined distance from the enemy. This distance is represented by the red circle in the picture above. If it is within the red circle, it uses varying power depending on the distance to the enemy. If it is closer, it will use more power because it has a higher chance of hitting the enemy. If it is farther away, it uses less power to conserve energy and also to slightly increase the chance of hitting since lower powered bullets travel faster.

## Performance
Currently, my robot performs well against all of the sample robots which are provided with the default installation of Robocode. Below is a table that summarizes the battle results of 100 rounds against each of the sample robots (excluding the interactive robots). The results shown below are fairly typical, but they ocassionaly vary by a few percent and one or two 1sts against a few of the robots.


| Enemy | My Score | Enemy Score | 1sts | 2nds |
|:------|:---------|:------------|:-----|:-----|
| sample.Corners | 17863 (92%) | 1527 (8%) | 100 | 0 |
| sample.Crazy | 15370 (99%) | 168 (1%) | 100 | 0 |
| sample.Fire | 17725 (100%) | 76 (0%) | 100 | 0 |
| sample.MyFirstJuniorRobot | 16639 (99%) | 238 (1%) | 100 | 0 |
| sample.MyFirstRobot | 16847 (98%) | 319 (2%) | 99 | 1 |
| sample.PaintingRobot | 16858 (98%) | 285 (2%) | 99 | 1 |
| sample.RamFire | 18104 (98%) | 389 (2%) | 100 | 0 |
| sample.SittingDuck | 18002 (100%) | 0 (0%) | 100 | 0 |
| sample.SpinBot | 14832 (96%) | 637 (4%) | 99 | 1 |
| sample.Target | 17872 (100%) | 0 (0%) | 100 | 0 |
| sample.Tracker | 17863 (92%) | 1527 (8%) | 100 | 0 |
| sample.TrackFire | 16216 (98%) | 361 (2%) | 100 | 0 |
| sample.Walls | 16394 (98%) | 282(2%) | 100 | 0 |
| sample.VelociRobot | 15665 (98%) | 393 (2%) | 100 | 0 |

None of the sample robots pose a significant threat to my robot. Because it uses an adaptive strategy (i.e. switching between movement strategies and varying its targeting strategy) my robot will behave slightly differently depending on the opponent. This allows it to perform well against a variety of movement and targeting strategies.

Currently I do not have any plans for major changes to my strategy. There is still room for improvements to its targeting and dodging, however, I am happy with its current performance given its relatively low complexity. There is also a lot that can be done to improve its ability to "learn" the best strategy between rounds. This is likely where I will focus any future work on this project.

## Testing
In my [last post]({{ site.baseurl }}{% post_url 2013-02-26-robot-testing %}) I explained some of the details of the tests that I developed for my robot. Since my strategy has not changed drasticaly since then, my previous tests have remained sufficient. Although my general strategy has not changed, I have made various changes and improvements to my robot. With a system like Robocode, each change has the potential to unexpectedly break the system. My tests proved effective in ensuring that my robot continues to behave as expected as I make changes.

I found the acceptance tests to be the most useful for this project. The results of the unit tests never change unless I specifically change the portion of the code that is being tested by the unit tests. For my robot, the results of the behavioral tests also rarely changed since I never made any drastic modifications to my strategy once I implemented the tests. Acceptance tests, on the other hand, were much less predictable. It is hard to know how a small change to a robot will affect the interaction between it and another robot. This caused my robot to ocassionally fail an acceptance test (i.e. fail to win a certain percentage of rounds against the enemy). Having an automated build system that would run these tests made it easy to see when a change would have a negative impact on my robot. Similarly, it also made it easy to see when a change improved my robot. This is something that behavioral and unit tests cannot accomplish.

## Reflection
This project has been very valuable to my growth as a software engineer. This is the first time that I've used an automated build tool and it has definitely sped up my development process. This is also the first time that I've used quality assurance tools. Although they could be annoying at times due to their constant complaining, they have definitely improved the quality of my code. Another important takeaway from this project was the importance of version control. At the start of this project, I didn't use any form of version control, which turned out not to be a great idea. A few times throughout the project, I found that I wanted to revert some of the recent changes that I've made.

If I were to do this project again, the main change I would make would be to use version control from the beginning. Although some of the tools that I used gave me some problems, their benefits greatly outweigh any of the issues that they caused. In general, for future projects I will definitely use all of the different types of tools that I have learned about throughout this project.


---
layout: post
title: Making a Mini Robot
categories: software-engineering
tags: java robocode
---
In my last post about [Robocode]({{ site.baseurl }}{% link _my_tags/robocode.md %}) I described my attempt at a competitive one-versus-one robot. In this post I talk about my attempts to shrink my robot so that I could submit it to the <a>RoboRumble</a> to see how it does against other robots from around the world.

## RoboRumble
RoboRumble is a system which attempts to provide rankings for Robocode robots. It relies on users to run battles. The results are then sent to a server and aggregated to provide a ranking for each robot. Submitting a robot to the RoboRumble is as simple as adding its name and a download link to a wiki page.

### Code Size
To make things more interesting, there are several weight classes which are based on the [code size](http://robowiki.net/wiki/Code_Size) of the robot. The The basic weight classes are shown in the table below.

| Class | Code Size |
|:------|:----------|
| Class | Code Size |
| General | No restrictions |
| MiniBots | 1499 bytes or less |
| MicroBots | 749 bytes or less |
| NanoBots | 249 bytes or less |

## Shrinking Redshift
Before making any changes to my robot, [Redshift]({{ site.baseurl }}{% post_url 2013-03-12-robocode-redshift %}), it had **2210 bytes** of executing code (which is how code size is measured). My goal was to make it as small as possible without having any impact on its behavior.

Actually, that's not quite true. The first change I made was to remove any code that had to do with melee battles. The RoboRumble is divided into one-versus-one and melee, so removing melee code would not affect its behavior in the one-versus-one RoboRumble. This change brought the code size down to **2126 bytes**.

The next change was to change static variables from private to public and remove the accessors and mutators associated with them. This had a small effect, bringing the code size down to **2109 bytes**.

Next I began to remove the utility classes which I had created, and add the code directly to the robot. The first one to go was the `AdvancedRobotUtility` which provided convenient methods to help the robot move and aim. It provided a number of methods that were not used by my robot, so I was able to reduce the code size by a significant amount. After removing the `AdvancedRobotUtility` the code size was **1866 bytes**. The next thing to go was the `MathUtility` class, which reduced the code size to **1641 bytes**.

The last utility class that I removed was the `BoundedQueue` which I used to keep track of the enemy velocity. This removed approximate 200 bytes from the code size which brought the size to **1434 bytes**, which is small enough to make it a MiniBot.

The last remaining utility class is the `RobotInfo` class. Instead of fully removing it, I decided to remove any unused code and make the instance variable public so that I could remove accessors and mutators. This reduced the code size to **1320 bytes**.

The last change that I have made is to remove debugging print statements, and the code to draw on the battlefield. This brought the final size to **1260 bytes**.

## Future Changes?
My next goal is to shrink my robot all the way down to a MicroBot, but that will likely prove to be a much more difficult challenge.

There is still some room to shrink the robot without affecting its behavior. The first thing I would probably do is to completely remove the `RobotInfo` class. I suspect this might reduce the size by an additional 20~50 bytes.

Next, there is maybe 50~100 bytes that can be shaved off by using some code reduction tricks. For example, performing long calculations on a single line instead of using variables to hold temporary values can save a few bytes.

Beyond that, it will probably require some modification of the robot's behavior. It is possible to drastically simplify the targeting. However, depending on how small the code needs to be, you may sacrifice some accuracy. I would suspect this to remove another 50~100 bytes.

This still leaves about 250~300 bytes before it is small enough to be a MicroBot. Finding those 300 bytes will definitely be the most difficult part. It will probably involve some significant changes to the robot's behavior. However, I hope to at least maintain some of its adaptive behavior.

## Reflection
So far, this has been a fun little project that is very different from the initial process of building my robot. When I first built the robot, I tried to focus on good software engineering practices and design. Everything was properly encapsulated. Functionality was nicely separated into methods and classes. However, in order to reduce the code size, I had to ignore many design principles that make code easier to maintain.

There is often a conflict between software engineering and performance. Software engineering practices are used to help build systems that are easy to test and maintain. However, in order to maximize performance, it is sometimes necessary to use clever hacks that defy basic software engineering practices. Making this robot seems to be a variation of that conflict.

The latest (smallest) version of my robot can be found on [GitHub](https://github.com/ttaomae/robocode-tkt-redshift/tree/codesize).


---
layout: post
title: Code Katas - Robocode
categories: software-engineering
tags: ics-691 java robocode
---
[Kata](http://en.wikipedia.org/wiki/Kata) is a Japanese term which is usually applied to martial arts. It refers to a series of choreographed movements used to practice good technique. More recently it has been used more generally and applied to many areas outside the realm of martial arts, such as software engineering.

## Code Katas
[Code kata](http://codekata.pragprog.com/) is a term coined by [Dave Thomas](http://pragdave.pragprog.com/) which describes programming exercises used to improve and hone skills. [Jeff Atwood](http://www.codinghorror.com/blog/2008/06/the-ultimate-code-kata.html) and [Steve Yegge](https://sites.google.com/site/steveyegge2/practicing-programming) claim that simply doing something does not count as practice. As an example, driving to work every day will not turn you into a professional driver. Similarly, simply writing code is not enough to make you a good programmer. You must engage in *effortful* study and practice in order to improve your skills. This is basis behind code katas.

## Robocode
[Robocode](http://robocode.sourceforge.net/) is an open source programming game where you develop a robot tank to battle against other robots. Each robot has three main components. They are the body, gun, and radar. The body carries the gun and radar. It can move forward or backward and turn left or right. The gun can turn and fire bullets. The radar can turn and detect other robots. On the surface it may seem simple, but a quick look at the [RoboWiki](http://robowiki.net/wiki/Main_Page) will reveal that there can be a lot of complexity involved in creating a good robot.

Before you even being writing any code, the first thing you must decide is what kind of robot you are creating. There are two common classes that are usually extended to create a robot. There is the Robot class and the AdvancedRobot class. There are more, but they are less commonly used, and I will probably not look into them. The Robot class is easier to use, but it has some limitations. The primary limitation is that it uses blocking calls. What that means in this context is that a Robot can only do one thing at a time. For example, it cannot turn while it is moving and it cannot move while it is turning its gun. An AdvancedRobot on the other hand can move, turn, turn its gun, fire its gun, and turn its radar all at once. This makes it much more powerful but also much more difficult to master.

After you've decided what kind of robot you are creating, you'll want to figure out how to make it move. In general, to do this, you need to override a number of different default methods. For example, the `run` method is the main method of the robot which will run once a battle begins. This is generally where you implement its basic behavior such as how it moves. There are also a number of methods of the form `onXXX` which are called after certain events happen, such as when you hit a wall, or when you scan another robot. This is how you react to things and make your robot more dynamic.

Once you start writing code, the complexities will start to reveal themselves. One of the first things you may realize, is that you need a decent understanding of some basic trigonometry to do a lot of useful things. For example, you may need to calculate how much you need to turn and how far you need to move in order to get to a certain location. Once you start trying to aim at other robots, the calculations start to get more involved. If you start to look into more advanced strategies, you'll come across terms such as [Anti-Gravity Movement](http://robowiki.net/wiki/Anti-Gravity_Movement), [Wave Surfing](http://robowiki.net/wiki/Wave_Surfing), [Anti-Surfer Targeting](http://robowiki.net/wiki/Anti-Surfer_Targeting), and [Neural Targeting](http://robowiki.net/wiki/Neural_Targeting). All of these are probably just as confusing as you think they are. Luckily for you and for me, I will not be getting into any of these advanced strategies today.

## Robocode Katas
This is not my first encounter with Robocode. I've played around with it several years ago. However, during my first encounter, I took more of a "learn as I go" approach. I would basically say, "I want my robot to do this," then I would read through the RoboWiki and other online resources and try to figure it out. Then I would move on and try something new. While this approached certainly worked -- that is, I was able to produce a functioning robot -- it definitely wasn't the best way to learn about Robocode.

Similar to the driving example above, simply creating a robot did not necessarily mean that I was actually learning how to become a good "Robocoder." This time, I took a more methodical approach. Before even considering how to create a competitive robot, I completed a number of [Robocode Code Katas](http://ics613s13.wordpress.com/modules/robocode/a13-robocode-code-katas/).

Specifically, I implemented the following robots:

> 1. **DefaultRobot:** The minimal robot. Does absolutely nothing at all.
> 1. **WallBangerRobot:** Move forward a total of 75 pixels per turn. When you hit a wall, turn right.
> 1. **SpiralRobot:** Each turn, move forward a total of N pixels per turn, then turn left. N is initialized to 15, and increases by a random value between 10 and 20 per turn.
> 1. **CenterSpinnerRobot:** Move to the center of the playing field, spin around in a circle, and stop.
> 1. **FourCornersRobot:** Move to the upper right corner. Then move to the lower left corner. Then move to the upper left corner. Then move to the lower right corner.
> 1. **CircleTheWagonsRobot:** Move to the center, then move in a circle with a radius of approximately 100 pixels, ending up where you started.
> 1. **FollowTheLeaderRobot:** Pick one enemy and follow them.
> 1. **FollowTheLeaderStealthilyRobot:** Pick one enemy and follow them, but stop if your robot gets within 50 pixels of them.
> 1. **RunAwayRobot:** Each turn, Find the closest enemy, and move in the opposite direction by 100 pixels, then stop.
> 1. **SimpleFiringRobot:** Sit still. Rotate gun. When it is pointing at an enemy, fire.
> 1. **ConserveBulletsRobot:** Sit still. Pick one enemy. Only fire your gun when it is pointing at the chosen enemy.
> 1. **ConservePowerRobot:** Sit still. Rotate gun. When it is pointing at an enemy, use bullet power proportional to the distance of the enemy from you. The farther away the enemy, the less power your bullet should use (since far targets increase the odds that the bullet will miss).
> 1. **TrackingRobot:** Sit still. Pick one enemy and attempt to track it with your gun. In other words, try to have your gun always pointing at that enemy. Don’t fire (you don’t want to kill it).

Given my previous experience with Robocode, after setting up my development environment, it only took a few minutes to refresh my memory and jump right into creating these robots.

Although my previous experience had mostly been with AdvancedRobot, for these code katas, I used Robot because it would allow me to learn the fundamentals more easily.

**1 - 3:** The first three robots were easy to create because they only used the basic features of a robot such as the `ahead` and `turnRight` methods and I only needed to override the `run` and `onHitWall` methods.

**4:** CenterSpinnerRobot is where I began to have some difficulties. I quickly realized that in order to move to a specific location I would need to use some trigonometry in order to figure out where I would need to turn to. After a few quick drawings and reminding myself of "SOH-CAH-TOA" I realized that I would need to use arctangent. After I implemented it, I started observing some weird behavior. It would only work if the robot was on the bottom half of the battle field. If it was on the top half, it would move in the exact opposite direction of the center. I realized that this must be some limitation of arctangent. After some Googling, I came across `atan2` which takes two arguments instead of one and would produce the results I was looking for.

**5:** Since I would need to turn and move to specific location for several robots, I decided to move that functionality into the DefaultRobot and extend from that instead of Robot. Given that I had already figured out how to move to a specific spot, moving to the four corners was easy. There was one small problem that I encountered. I was trying to move to the exact corner of the battle field, but since the robot has some size, the center of the robot (which is where its location is considered to be) could not reach the corner. In order to combat this, instead of moving to the corner, I moved to a point 50 pixels away from the corner. The number 50 was arbitrarily chosen as somewhere that was close to the corner, but not touching it. I later modified this to be half the length/width of the robot away from the corner.

**6:** Again, I encountered some problems with this robot. At first, I wasn't sure how to approach the problem. I wasn't sure if I should use some trigonometry to calculate angles and distances to travel. I ended up taking a much simpler approach. I simply calculated the circumference of the circle, which would be the total amount that I should travel and I divided this into a number of iterations. Each iteration, I would turn a fraction of 360 degrees, and move forward a fraction of the total circumference. It wasn't perfect, but it got the job done.

**7-8:** These were the first robots which used the radar. My previous experience with Robocode came in handy here. I already knew I would need to override the `onScannedRobot` method, so it was easy to implement both of these.

**9:** For me, this was the most difficult robot to implement. First I needed to decide how to keep track of the robot distances. I decided on a map which would take a robot name as a key and map to a robot distance. Once I scanned all robots, I would select the closest one and turn away from that one. I soon realized, I would also need to keep track of a bearing to that robot. In order to do that, I wrote a small class called RobotInfo which would hold a distance and a bearing. Now the robot name would map to an instance of RobotInfo instead of just a distance. After that was sorted out, it was simple to go through the map and find the closest robot after each time that I scanned all robots.

**10-13:** Again, my previous experience with Robocode was very helpful while implementing these robots. I did not encounter any problems with these katas because the calculations were familiar to me.

## Reflection
Despite the fact that I have used Robocode before, I was still able to learn from this experience. Clearly, these katas were created with a beginner in mind, but I did still find some challenge in them. They were also able to give me some ideas for building a competitive robot. Specifically, finding the closest robot and possibly keeping track of all other robots is something that I hadn't considered before. I could use this to find the best target or move away from potential threats.

Although this probably wasn't the best circumstance for my first code katas, I still found the experience to be useful. In general, they seem like a great way to develop programming and software engineering skills. Even doing simple tasks with something you're familiar with can improve your skills. By forcing yourself to engage in these seemingly simple tasks, code katas can also be a great way to develop discipline. You could also make it more engaging by providing several different implementations of the same thing. This will force you to think out of the box and perhaps learn a new technique or skill that you had never considered before.


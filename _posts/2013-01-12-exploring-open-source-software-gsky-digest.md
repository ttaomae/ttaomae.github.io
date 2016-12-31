---
layout: post
title: Exploring Open Source Software - gSky Digest
categories: software-engineering
tags: ics-691
excerpt_separator: <!--end_exceprt-->
---

[The Three Prime Directives of Open Source Software Engineering](http://ics613s13.wordpress.com/course-info/the-three-prime-directives-of-open-source-software-engineering/) are as follows:

1. The system successfully accomplishes a useful task.
2. An external user can successfully install and use the system.
3. An external developer can successfully understand and enhance the system.

My goal for today was to find a Java-based open source project that is of interest to me and that I could potentially contribute to. After finding such a project, I would explore how well it meets the Three prime Directive of Open Source Software Engineering.
<!--end_exceprt-->

[<img src="gsky.png" alt="gsky" />](gsky.png)

After browsing through some of the different categories at [SourceForge](http://www.sourceforge.net/), I decided to take a look at [gSky Digest](http://gskydigest.sourceforge.net/). gSky Digest is a lightweight application that notifies the user of various astronomy events ranging from the phases of the moon to the [apogeum](https://en.wiktionary.org/wiki/apogeum) and [perigeum](https://en.wiktionary.org/wiki/perigeum) of the various bodies of the Solar System. Currently it only has support for the Sun, the Moon, and the seven planets (excluding Earth). However, the goal is to provide information for all events that can be observed by the naked eye or using binoculars.

Overall, I found that gSky Digest satisfies the first two Prime Directives fairly well. Of course, that is not to say that there are no problems. Its greatest shortcoming is in satisfying the third Prime Directive, which I would argue is that most difficult to accomplish.

## Prime Directive # 1
Does this system successfully accomplish something useful? Absolutely. The information provided by this application can be useful to amateur astronomers who are new to the subject. As the application is developed and it provides more information, it will be more useful to those who have more experience in the area. For me, it is a simple way to see when there will be a new moon, or when the moon will or will not be in the sky. This is important for taking good photographs of the night sky because the light from the moon interferes with capturing light from the stars.

## Prime Directive # 2
Can an external user install and use the system? Sure. I did. This system satisfies the second Prime Directive, but there is definitely room for improvement. The installation is very simple and there is little room for improvement there. Using the application is, in my opinion, fairly simple and intuitive.

However, one issue that I came across was during my very first interaction with the program. During the first launch of the program, you must select your location. In order to do this you must scroll through a very long list of cities. I later found that you can begin typing and it will jump to the first city that matches your typing. However, there is no indication that this will happen and I did not see any search feature.

Another potential issue is that there is no explanation for some of the information provided. For example, a user who is not familiar with using arcminutes and arcseconds to measure angles would likely be confused by this program. There are also a number of astronomy terms, such as transit, conjunction, and azimuth, that will be unfamiliar to the average person. However, in the case of this particular program it is not a huge issue because the typical user of this program will likely be interested in astronomy and be familiar with many of these terms.

## Prime Directive # 3
Can an external developer successfully understand an enhance the system? This is where the system begins to fall short. This system's greatest asset towards satisfying the third Prime Directive is that it is very simple to build the program from the source which means it is easy to be able to see the effects of any changes you make to the source. Included with the source files is a set of very simple instructions for running the program from the source or building your own executable. After installing the appropriate software, which includes the <a title="JDK" href="http://www.oracle.com/technetwork/java/javase/downloads/index.html">JDK</a>, <a title="Eclipse" href="http://www.eclipse.org/">Eclipse</a>, <a title="SWT for Eclipse" href="http://www.eclipse.org/swt/">SWT for Eclipse</a>, and <a title="NSIS" href="http://nsis.sourceforge.net/Main_Page">NSIS</a>, it took only a few minutes to create my own executable. Considering that I already had the JDK and Eclipse installed, the entire process took no more than 20 minutes.

Unfortunately, this system does not do much more to help satisfy the third Prime Directive. The code seems well organized, but there are limited comments in the code and I could not find any documentation that would explain the structure of the code. There was however a document with a list of known bugs and features to implement. This would give new developers an idea of where to begin, but they would still have to figure out the structure of the code on their own before they can begin making any changes.

Overall, this system doesn't quite satisfy the third Prime Directive. However, with some time and a little perseverance it probably would not be extremely difficult to contribute to this project.


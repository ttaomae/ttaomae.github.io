---
layout: post
title: '"Bug" fix: JVM and the Garbage Collector, or "Why is the JVM using all cores
  at 100% and making no progress?"'
---

In my [last post]({{ site.baseurl }}{% post_url 2014-08-07-solving-the-tetris-cube %}) I discussed my journey towards solving a puzzle using programming. It was quite a long post so I decided to leave out some details and one particular roadblock that I ran into.

Before I continue I want to clarify why "bug" is in quotes in title. This isn't really a bug in the sense that there is an error in my code. And it certainly isn't a bug in the JVM or garbage collector as might be implied by the title. Instead it's more of a story about what I thought might be a bug but turned out to just be me butting heads with the JVM.

## The Problem
If you read my last post you'll recall that my final solution took about an hour to complete using 8 threads (4 cores with Hyper-threading). Of course it took even longer when I was using less threads (so that I could still use my computer without it being completely bogged down by the program). Imagine my dismay when I found out that after it was about 80% complete, it would seemingly stop making progress and run all cores up to 100%, regardless of how many threads I was running.

## Debugging

### Try, Try Again
I had never really seen behavior like this so I didn’t exactly know where to begin. My first thought was to just try running it again and hope that it might somehow resolve itself. This was more wishful thinking than a serious attempt at resolving the issue, but I thought it was worth a shot anyway. Next, I tried running it from the command line instead of from Eclipse thinking that maybe there might somehow be a problem with Eclipse’s console. Again, this was mostly wishful thinking. Or maybe it was me in denial that there might be a problem with my code.

### Parallel Problems?
My next thought was that maybe there was a problem with the multithreaded code since everything seemed to be working (on the smaller problems) prior to adding the multithreaded code. Writing concurrent programs is notoriously difficult and riddled with pitfalls, however I wasn’t completely convinced that that was the issue. Not because I’m great at writing concurrent code, but because the behavior didn’t quite fit with the common problems that arise, such as [race conditions](http://en.wikipedia.org/wiki/Race_condition# Software), [starvation](http://en.wikipedia.org/wiki/Resource_starvation), and [deadlocks](http://en.wikipedia.org/wiki/Deadlock) since these would not, to my knowledge, cause the CPUs to ramp up. A [livelock](http://en.wikipedia.org/wiki/Deadlock# Livelock) might have been a possibility but it still didn’t quite make sense. Every time I ran the program it would get stuck at around the same spot regardless of how many threads I used. With the issues I mentioned above you would expect more random behavior, such as getting stuck at different points in the computation rather than always around 80%. It also didn’t make sense that it would suddenly start to use the CPU more. For example, if I used 4 threads, I would expect the CPU usage to max out around 50% since I am only using half of the 8 virtual cores. Instead it would jump up to 100%.

I tried investigating my multithreaded code, but I couldn’t see anything obviously wrong. For the reasons mentioned above it didn’t seem likely to be the cause of the problem, so I decided to move on.

### Implementation Issues?
My next guess was that maybe I had implemented the algorithm incorrectly. Luckily I wasn’t worried that the algorithm itself was wrong since it was written by [Donald Knuth](http://en.wikipedia.org/wiki/Donald_Knuth). However, it was certainly possible that I had messed up somewhere in the implementation. I went through my code several times comparing my code to Knuth’s pseudocode and I found no issues. Unfortunately I didn’t have a strong point of comparison for the multithreaded algorithm, but I went through it again but still didn’t see anything wrong. I didn’t expect to make much more progress so I moved onto my next idea.

### More Memory?
At this point I decided to focus on the issue of the 80% completion. What was it about 80% that would make the program suddenly stop working? Another key number that I had missed previously was the approximately 3 gigabytes of memory that were being used every time it stopped working. I must have overlooked this as a possibility because every other time I’ve had memory issues in Java I would get an `OutOfMemoryError`. Although I wasn’t quite sure why I wasn’t getting an `OutOfMemoryError`, this seemed like a likely culprit so I started to mess around with the JVM settings.

## Solution
You can get a list of JVM settings by typing `java -help` on the command line or `java -X` for non-standard options. Since I already suspected that memory was the issue I started with the `java -Xmx` option, which allows you to set the maximum heap size. For example, if I wanted to only allow a maximum of 100 megabytes, I could use the option `java -Xmx100m` or `java -Xmx100M`. Similarly, you can use *k* or *K*, and *g* or *G* for kilobytes and gigabytes, respectively. You can also just specify a number of bytes without a letter at the end.

By going with a smaller heap size I expected to run into issues much quicker, which is exactly what happened, which confirmed that it was a memory issue. Although I could leave it at that and simply increase the heap size and be done with it, I wasn’t quite satisfied with this explanation. I still didn’t understand why I wasn’t getting an `OutOfMemoryError`, which would have saved me a lot of frustration from the beginning.

I looked more into the options and decided that the `java -verbose:[class|gc|jni]` option might be useful. Being that the issue was related to memory, the `gc` (which stands for [garbage collection](http://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29)) option was the obvious choice. What I found was that near the end, when the program seemingly stopped making progress, the garbage collector (GC) would repeatedly run but free up very little memory.

It seems that even at the end there is a small amount of memory that can be freed, which prevents the `OutOfMemoryError` from being thrown. Presumably the program is actually still making very slow progress. Each time the GC runs it frees up some memory which is quickly used up by the program resulting in the GC running again. This cycle slows the progress of the program while also causing the GC to ramp up the CPU usage.

## Reflection
This was quite a different “bug” than what I am normally used to. Being that it was a fairly complicated process that only seemed to have problems after long computations, using a debugger was a little out of the question. It would be difficult to find a good place to set breakpoints. Even if I were able to, stepping through the code and trying to understand everything would be a nightmare with the number of nodes and references there were (if you recall, the Dancing Links technique uses “double circular doubly linked lists” as I called them). In the end it turned out the be a good thing that I didn’t use a debugger since the problem wasn’t really related to my code at all.

I hope for this to the first of a sort of series of posts. They will tend to be shorter than most of my previous posts and will focus on a single bug fix rather than an entire project.


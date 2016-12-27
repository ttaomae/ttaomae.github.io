---
layout: page
title: Pseudo-n-body Problem
excerpt_separator: <!--end_excerpt-->
---
[<img src="{{ site.baseurl }}/files/nbody.png" alt="n-body" />]({{ site.baseurl }}/files/nbody.png)

This is a project that I worked on during my last semester of my undergraduate studies. It was an individual project under faculty supervision. My goal was to introduce myself to general-purpose computing on graphics processing units ([GPGPU](http://en.wikipedia.org/wiki/GPGPU)). For this project I used [OpenCL](http://en.wikipedia.org/wiki/OpenCL).
<!--end_excerpt-->

The problem which I was attempting to solve was a variation of the <a href="http://en.wikipedia.org/wiki/N-body_problem">n-body problem</a>. In the traditional n-body problem, the motion of a group of point masses is calculated based on their gravitational interaction with each other. In this variation, there are a number of "pseudo-particles" which do not move and there is one or more "test particles" which move based on their interaction with the pseudo-particles, but not with other test particles.

First, I wrote a version which only uses one test particle. I wrote a single-threaded CPU version in C, as well as a GPU version in OpenCL. I found that, at worst, the GPU version performed comparably to the CPU version. I also found that if there were more pseudo-particles, the GPU version performed better. At best, the GPU version completed in about half the time of the CPU version.

I also wrote a version which can use a variable number of test particles. Again, I wrote a single-threaded CPU version and a GPU version. This time I found that the GPU version performed the same or worse than the CPU version.

There are still a lot of optimizations that can be done on the GPU version. However, the same can be said of the CPU version, so it is hard to say, without further development, whether or not the GPU can provide a significant performance boost for this problem.

The source code can be found [here](https://github.com/ttaomae/pseudo-n-body/).

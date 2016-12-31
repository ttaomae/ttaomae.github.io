---
layout: page
title: Gesture Recognition with the Microsoft Kinect
date: 2013-12-23 16:10:53.000000000 -10:00
---
For this project, my partner and I explored a small subset of the capabilities of the Microsoft Kinect. Specifically, we used the skeletal tracking feature to detect gestures which are then used to control presentation software.

## Gesture Recognition
[<img src="{{ site.baseurl }}{% link files/skeleton_example.png %}" alt="skeleton example">]({{ site.baseurl }}{% link files/skeleton_example.png %})

In this project we outline the importance of developing good gestures when they are used in support of a different, primary activity. When the gestures are secondary to a primary activity, they must be distinct from motions or actions that might be performed during the primary activity in order to reduce false detections.

We developed five different gestures that are used to perform various actions in a presentation software such as Microsoft PowerPoint. The actions that we decided on were forward or backward one slide, forward or backward five slides, and a single gesture to start or stop the presentation. To go forward or backward one slide the user makes a clockwise or counter-clockwise circle with their hand. To move by five slides the user performs the same gesture along with holding their left hand to the side of their head. The start and stop gesture is performed by placing both hands to the side of the head for at least one second.

## Results
We conducted a test with approximately forty users to test the accuracy of our gesture detection. Each user also completed a survey giving feedback on the usability and enjoyment of the gestures. There is still room for improvement of both the accuracy and usability of the gestures, but our results so far have been fairly positive. The accuracy for the various gestures averaged around 85 percent and the surveys indicated that the users generally had a positive experience with the gestures.

For a more information about the project including more detailed discussions of the methods and results, you can read my [blog post]({{ site.baseurl }}{% post_url 2013-12-23-gesture-recognition-with-the-microsoft-kinect %}) about it or you can read [the paper]({{ site.baseurl }}{% link files/gesture-recognition-with-the-microsoft-kinect.pdf %}).

---
layout: post
title: Gesture Recognition with the Microsoft Kinect
categories: computer-science
tags: human-computer-interaction computer-vision
---
With recent advances in technology, new methods of computer interaction are becoming available. In this blog post I will briefly describe my investigation of gesture recognition using the [Microsoft Kinect for Windows](http://www.microsoft.com/en-us/kinectforwindows/) as a means of controlling software. For a much more detailed description of the methods and results of this project, you can read [the paper]({{ site.baseurl}}/files/gesture-recogntion-with-the-microsoft-kinect.pdf).

## The Kinect
The Microsoft [Kinect](http://en.wikipedia.org/wiki/Kinect) was originally developed for the Xbox 360, however, when Microsoft saw the potential among researchers and hobbyists, they released a version for Windows. The Kinect has three main components: an RGB camera, a depth sensor, and a multi-microphone array. The Kinect SDK provides easy access to many of the Kinect's capabilities such as voice recognition, acoustic source localization, face tracking, and skeletal tracking, which was used for this project. The skeletal tracking feature provides the 3D coordinates of 20 "joints." They are the head, shoulder center, spine, hip center, and the left and right shoulders, elbows, wrists, hands, hips, knees, ankles, and feet.

## Gesture Recognition
In this project, we attempt to detect several gestures in order to control presentation software, such as Microsoft PowerPoint. We developed the gestures to be distinct from natural movements that may occur during a presentation, but also easy to perform both in terms of accuracy and physical difficulty.

### Developing Good Gestures
Although it wasn't our original goal, it quickly became apparent that developing a good gesture is crucial to effective gesture recognition. This is especially true when the gestures are in support of a different, primary activity. In this case, giving a presentation is the primary activity and using gestures to control the presentation software is secondary to that activity. This is in contrast to using gestures to control a game, for example, where performing gestures to play the game is the primary activity.

We created the following general guidelines for developing gestures which are used to support a primary activity:

* Distinct from common motions used in primary activity.
* Natural and pleasant to use. Not physically difficult to perform, particularly if it is a common gesture.
* Minimal precision required to perform gesture.
* Easy to remember. Good mapping from gesture to its associated action.
* Related actions, such as forward and backward, should have related gestures.
* Relatively easy to detect.

### Selected Gestures
We developed gestures to perform several different actions. They are forward or backward one slide, forward or backward five slides, and a single gesture to start or stop the presentation. The forward one slide gesture is a clockwise circle performed with the right hand, centered at a point to the right of the user's waist. The backward one slide gesture is the same except that it is a counter-clockwise circle instead. The forward and backward five slide gestures are the same gestures as their one slide counterparts with an added modifier of holding the left hand up to the side of the head. The start and stop gesture is performed by holding both hands up to the side of the head for at least one second.

Our selected gestures are certainly not ideal and do not provide the most natural mapping between the gesture and the corresponding action. However, we found that it was much more important to find a gesture that would cause minimal false detections and would not interfere with the presentation. Having said that, there is still room for refinement of the gestures.

### Gesture Detection
Our gesture detection is performed using an algorithmic method. Although this method is not as effective for large sets of gestures or for complex gestures, it served us well since we had only a few simple gestures. It also has the advantage of having no up front work, unlike other methods such as artificial neural networks.

In order to limit the precision required by the user performing the gesture and to make detection easier, we do not actually detect a circle for the forward and backwards gestures. Instead, we track the hand as it travels through four quadrants. If the hand travels through all quadrants, in order, then returns to the original quadrant, we say that the user has performed a circle.

Tracking a hand to the side of the head is done by keeping a history of hand positions for a limited number of frames and finding the average height over the history. This approach has two advantages. First, it prevents situations where the Kinect does not accurately track the hand for a single frame, which could potentially cause it to be outside the acceptable range of heights. The second advantage is that if you keep a long enough history, you can impose a time limit, which is what was done for the start and stop gesture.

The following image shows an example skeleton as well as the information we used in detecting our gestures. The green dots and lines are the skeleton provided by the Kinect. The blue lines show the four quadrants which the user's hand must move through in order to perform a circle gesture. The cyan lines show the range of acceptable heights for the hand to be considered to the side of the head.
[<img src="skeleton_example.png" alt="skeleton_example" />](skeleton_example.png)

## Results
To test our gesture recognition, approximately forty users were asked to perform the various gestures while we recorded the accuracy of the software. After the test, each user completed a survey on the usability of the gestures. The results were generally positive, but there is still room for improvement of the gestures, the software, and the testing. The following table shows the results of the accuracy of our gesture detection, where the first column is the gesture we asked the user to perform and a "(3)" indicates that they were asked to perform it three times in a row.

| Gesture | Average attempts | % accuracy | % reminder/clarification |
|:--------|:-----------------|:-----------|:-------------------------|
| Start | 1.037 | 96 | 13 |
| Forward | 1.453 | 69 | 52 |
| Forward (3) | 3.377 | 89 | 0 |
| Backward | 1.151 | 87 | 13 |
| Backward (3) | 3.415 | 88 | 0 |
| Forward Five | 1.302 | 77 | 30 |
| Forward Five (3) | 3.509 | 85 | 0 |
| Backward Five | 1.283 | 78 | 0 |
| Backward Five (3) | 3.642 | 82 | 0 |
| Stop | 1.094 | 91 | 16 |

The following charts provide a summary of the survey results.


[<img src="gesture_survey_results.png" alt="gesture survey results" />](gesture_survey_results.png)

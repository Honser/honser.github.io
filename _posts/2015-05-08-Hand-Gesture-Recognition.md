---
layout: post
title:  "Hand Gesture Recognition"
date:   2015-05-08 19:00:00 -0600
categories: jekyll update
---
	| C++ | OpenCV |

>Source : <https://github.com/Honser/Hand-Gesture-Recognition>  


--- 
#### Objectives
- Understand how OpenCV operate.
- Make it detect and count number of fingers.
- Simulate the program.
&nbsp;


---
#### Operation
1. Capture the video frame.
2. Convert the frame into gray scale.
3. Segment the hand in the image using color based method.
4. Obtain the binary image of the hand portion after segmentation.
5. Use morphological thinning on binary image.
6. Count the number of end points in the thinned image.
7. Display it.
&nbsp;

---
#### Simulation Result  

![result]({{ site.baseurl }}/images/5F.png)

![result]({{ site.baseurl }}/images/1F.png)

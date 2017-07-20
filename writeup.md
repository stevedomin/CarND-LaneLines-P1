# **Finding Lane Lines on the Road**

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[original]: ./test_images/solidWhiteCurve.jpg "Solid White Curve"
[grayscale]: ./debug_output/grayscale.jpg "Grayscale"
[gaussian]: ./debug_output/blurred.jpg "Gaussian blur"
[canny]: ./debug_output/canny.jpg "Canny Edge detection"
[region]: ./debug_output/masked.jpg "Region of interest"
[hough]: ./debug_output/hough.jpg "Hough transform"
[final]: ./test_images_output/solidWhiteCurve.jpg "Final"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps:

1. I converted the image to grayscale. This step is important for a processing step later in the pipeline: Canny Edge
detection.

2. After that, I applied a Gaussian Blur with a kernel size of 3. The Canny algorithm provided by OpenCV already does
that internally.

3. I apply the Canny algorithm to detect the edges in my image. This algorithm by detecting strong edges (strong
gradient) pixels above a specified `high_threshold`, and reject pixels below `low_threshold`. It will then include pixels
between `high_threshold` and `low_threshold` if they are directly connected to strong edges.

4. The next step is to define a region of interest. Because we know that the camera capturing the video is at a fixed
position we can determine an area of interest using a 4-sided polygon starting from the bottom of the image. Every edge
outside of this region will be discarded.

5. Finally, we apply a Hough Transform to detect line segments from edges detected by the Canny detector inside our
region of interest.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function this way:

6. First, I iterate over each line segments extracted by the Hough Transform step and fit a 2-degree polynomial on each
extremity of the segment.

7. I then reject all segments where the slope is not matching certain conditions. We know we don't want any line segment
that is roughly horizontal for instance so there is no need to include them in our calculation.

8. Then, based on the slope, I sort segments into two arrays dependening on whether they belong to the right or the left
lane.

9. Once I have all the segments for a given lane in the image, I compute the mean of all these line segments and use the
slope and y-intercept to extrapolate the rest of the line.

10. I then draw the extrapolated line for each lane, and stores the polynomial coefficients so that I can use them on the
next frame to smooth the transition between each frame.

11. Finally, I merge the image with the extrapolated lane lines onto my original image.

Here are the images at each step of my pipeline:

Original image:
![Original][original]

Grayscale:
![Grayscale][grayscale]

Gaussian blur:
![Gaussian blur][gaussian]

Canny Edge detection:
![Canny Edge detection][canny]

Region of interest:
![Region of interest][region]

Hough transform and lane extrapolation:
![Hough Transform][hough]

Final image:
![Final][final]

### 2. Identify potential shortcomings with your current pipeline

There are several shortcomings with the current version of my pipeline:

* I am not doing any kind of color selection so in tougher environment I expect that my pipeline would not work so well.
We can already see some limitations on the challenge video.

* The smoothing algorithm only takes into account the coefficients from the last frame so it's not as smooth as it could
be, there's still a bit of jumpiness.

### 3. Suggest possible improvements to your pipeline

A simple improvement would be to store all past coefficient values and use a weighted average to compute the new
coefficient.

A possible improvement would be to apply some kind of horizon detection so that we don't rely on hardcoded value to
specify the region of interest. This mean the pipeline would work well in slopes.

Another potential improvement could be to detect the curves of the lanes. Right now my pipeline can only draw straight
lines and that would not work well at all on roads with a lot of turns.



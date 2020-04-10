# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[grayscale]: ./write_up_images/grayscale.jpg "Grayscale"
[gaussian]: ./write_up_images/gaussian_blur.jpg "Gaussian Blur"
[canny_edge]: ./write_up_images/canny_edge.jpg "Canny Edge"
[roi]: ./write_up_images/roi.jpg "Region of interest"
[hough_lines]: ./write_up_images/hough_lines.jpg "Hough Lines"
[result]: ./write_up_images/result.jpg "Result"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.
The pipeline consists of five steps: convert to grayscale, Gaussian blur, Canny edge detection, region of interest cropping,
and Hough line transformation.

#### Convert to grayscale

In the first step, we convert the image to grayscale, this is standard practice when applying Canny edge detection. 
Although, variations on the algorithm designed to work with colour images do exist.

![alt text][grayscale]

#### Gaussian blur

The Gaussian blur step convolves the image with a Gaussian kernel, to remove noise from the image. This is key, as the
Canny edge algorithm is susceptible to noise. However, you can see that the kernel also softens edges in the image, and 
so must be used with care as to not blur the lines into the road completely, this can happen if large kernel size is 
picked. It is for this reason, a small kernel was chosen, of only five pixels, so as to remove some noise, but preserve
the lines.

 ![alt text][gaussian]

#### Canny edge

Next, the Canny edge detection method was applied. This computes gradients in the image, where parts of the image
with the sharpest gradients are strong edges. The core of the algorithm is a convolution with both an x-directional, 
and a y-directional gradient kernel. This can then be used to converted to calculate direction of gradient, and 
absolute size of gradient. As part of the algorithm, a max threshold and min threshold are required. Anything, that is
a very stong gradient with intensity greater than the max threshold, is seen as positively an edge; anything, below the
 lower threshold, is definitely not an edge. For anything in between, whether it is an edge it determined as to whether 
 or not it is connected to a gradient higher than the maximum threshold (a positive edge).
 
 The results can be seen below, we can see that the road marking are clearly visible.
![alt text][canny_edge]

#### Region of interest

Generally, any edges outside the road surface will be input noise to later parts of the pipeline. To counter this,
we apply a region of interest mask, that only leaves the lines on the road.
![alt text][roi]

#### Hough lines

Once complete, we calculate a Hough transformation to find lines in the image. This is achieve by creating a grid in Hough
space, whose dimensions are angle and distance, and are denoted theta and rho respectively. If we imagine a line in Cartesian 
space, theta is the angle a perpendicular line makes with the origin, and rho is the perpendicular distance to the origin.
An accumulator grid is formed in Hough space, an for each pixel in the Canny image, possible lines are detected in the 
local neighbourhood of each pixel. Every time a line is found, the grid position in Hough space is increased. Eventually,
the grid positions in Hough space with the highest accumulation become the most likely lines.

We must choose parameters to make the algorithm effective: the first is the granularity of the grid in Hough space, we 
use a single pixel, and a single degree; the second, is minimum line length, below which line segments are rejected from
accumulating in Hough space; and the third, max line gap, is the maximum distance between points considered to be on the
same line.

Once, the candidate lines are found in Hough space, a line of best fit is fitted to be able to extend from the top of the
lane to the bottom. First, the candidate lines are split into left and right hand lines. This is using the fact that the 
lines of the left have positive slope, and the lines on the right have negative slope. Then standard formulas for the
line of best fit are used to calculate the lane lines. The results are shown below.

![alt text][hough_lines]


### 2. Identify potential shortcomings with your current pipeline

There are several shortcomings in this pipeline:

1. The region of interest is fixed arbitrarily on the video, and relies on the lines being in roughly the same 
place. However, if the car turns, or crosses lane this breaks down.
2. It can get confused with lines on the opposite side of the highway, if these fall in the region of interest.
3. It cannot deal with shadows and different lighting conditions well.
4. It assumes that the lines are straight, this is a problem when the curve of the road is very sharp.
5. If there is heavy traffic, and the lane lines may not visible far from the car.

### 3. Suggest possible improvements to your pipeline

The are a few improvements that could be made:

1. Use colour information to help distinguish lane lines.
2. Fit curves of best fit to the lines, to find curvature better.
3. Have different levels of lane approximation. One of lines close to the car, and several moving away from the car.
This could be used in heavy traffic, but would give a more accurate idea of where we are, and where we are going to go.
4. Use prior knowledge of the lane lines to estimate approximate ROI for the next frame.

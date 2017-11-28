# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on my work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[image2]: ./examples/gaussian.jpg "Gaussian"
[image3]: ./examples/canny.jpg "Canny"
[image4]: ./examples/interest.jpg "Interest"
[image5]: ./examples/hough.jpg "Hough"
[image6]: ./examples/final.jpg "Final"
---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps. 

First, I converted the images to grayscale 

![alt text][image1]

After that I applied gaussian blur to the grayscaled image. I used a kernel of size 5.

![alt text][image2]

After that I applied canny edge detection with low threshold of 50 and high threshold of 150, this step detects a good set of edges, mostly picked this from the in course quiz.

![alt text][image3]

After this I applied the region of interest filter, to just have the edges detected for lane lines.

![alt text][image4]

After identifying the region of interest, I ran the hough line detection, using the lines detected I was able to draw the following lines. 

![alt text][image5]

Following the hough line detection step I created a weighted image with the above image and original image to create the final image as displayed below.

![alt text][image6]

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by the following steps

```
#I created two set of lines
left_lines = [] # all lines with slope > 0.5
right_lines = [] # all lines with slope < -0.5

# After this I created 4 set of points

right_x = []  # all x coordinates of end points of right lines
left_x = []  # all x coordinates of end points of left lines
right_y = [] # all y coordinates of end points of right lines
left_y = []  # all y coordinates of end points of left lines

# Using these points I ran linear regression to find the average line
line_left = np.polyfit(np.array(left_x), np.array(left_y), 1)
line_right = np.polyfit(np.array(right_x), np.array(right_y), 1)

# Also identified the y high and y low , based on the imshape for y low 
# and the highest point on left or right lines for y high
y_low = img.shape[0]
if len(right_y) == 0:
    y_high = np.min(left_y)
elif len(left_y) == 0:
    y_high = np.min(right_y)
else:
    y_high = np.min(np.array([np.min(right_y), np.min(left_y)]))
# Using these I identified the x coordinates
x_left_low = (y_low - line_left[1]) / line_left[0]
x_left_high = (y_high - line_left[1]) / line_left[0]
x_right_low = (y_low - line_right[1]) / line_right[0]
x_right_high = (y_high - line_right[1]) / line_right[0]

# using these points I was able to draw the 2 lines
cv2.line(img, (int(x_left_low), y_low), (int(x_left_high), y_high), color, 2* thickness)
cv2.line(img, (int(x_right_low), y_low), (int(x_right_high), y_high), color, 2* thickness)

# Then just returned a weighted image for transparency in the lines
return weighted_img(img, original_img, α=0.8, β=0.5, λ=0. )
```


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be when a line in between the two lane lines with a slope greater then 0.5 or less then -0.5 would be present, the linear regression will fail and cause the fitted line to be totally wrong.  

Another shortcoming could be if the lane lines are a little out of the region of interest it might result in blank line on the final image.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to eliminate lines identified in the middle area of the image , so that linear regression works.

Another potential improvement could be to find a better way then liner regression to draw lines.

Another improvement could be to find a even better way to define the region of interest. The current way seems to be too much brute force.

# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/1.png "Pipeline"
[image2]: ./examples/2.png "Grayscale"
[image3]: ./examples/3.png "Gaussian Blur"
[image4]: ./examples/4.png "Canny Edges"
[image5]: ./examples/5.png "Region of Interest"
[image6]: ./examples/6.png "Averaged lane lines (over Hough lines)"
[image7]: ./examples/7.png "Final Image"

---

### Reflection

My approach is a straightforward combination of the pipeline we saw in class and extrapolating straight lines by using average slopes. The only extension is that I use information from past frames as well.

Even though video processing is essentially image processing, i.e. it goes frame by frame, valuable information can be gained from looking at the near history of frames. Movements are usually smooth enough for this type of data to be useful. It can also help to generalize the algorithm to trickier lightning situations, varying colors such as in challenge.mp4 and reduces overall jitter in line overlay. 

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

1. I initialize an empty history (global variable).
3. First, I grayscale the image so we can focus on only one channel.
4. Next, I use a preliminary Gaussian blur with kernel = 5.
5. I then use Canny edge detection with parameters = [50, 150]  (1:3 ratio as author suggested) to detect edges based on the gradients.
6. Thereafter, I extract the Region of Interest (RoI). I use a trapezoid shape in the bottom half of the picture which I parametrize by 3 variables: top_padding bottom_padding, height. Padding indicates the horizontal distance from edges of the image as percentage of image width. For example, top_padding = 0.5 and bottom_padding = 0 creates a triangle shaped trapezoid, while top_padding = 0 and bottom_padding = 0 creates a rectangle with the width of the image. 
7. I then extract straight lines from my Canny-detected edges in RoI using Hough transforms with parameters (rho=2, theta=1rad, threshold=15, min_line_len=30, max_line_gap=40). 
8. Using the modified ````draw_lines()```` function, I do the following: 
    *  Looping over Hough lines, I calculate the slope and intercept for each line
  * I filter out lines with slopes outside of reasonable values (slope_limit variable)
  * For each suitable line, I separate lines belonging to left or right lane and check if it fits the intercept limits (left_lane_intercept_limit and right_lane_intercept_limit, expressed as a percentage of the image width)
  * I then calculate the length (using hypothenuse formula) and save the slope, intercept and length of the line
  * I calculate the weighted average slopes and intercepts, using lengths as weights and save the line coefficients to history. (````get_history()````)
  * I read the last 15 (hisotry_length variable) slopes and intercepts from history and calculate their unweighted average. This defines the lane lines drawn on the images
  * I extrapolate to the bottom and top of my RoI, using algebraic expressions for the x values, i.e. if y_roi_bottom_left is the endpoint of the line, the x-value is given by: x_roi_bottom_left = (y_roi_bottom_left - intercept_left)/slope_left. 
  * I plot the resulting lines on a black canvas and use ````weighted_img```` to add the transparency to original image.

#### Visualization

![alt text][image1]
![alt text][image2]
![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]


### 2. Identify potential shortcomings with your current pipeline

#### Major shortcomings
1. Lanes are only detected if the slopes are in "reasonable" ranges. For the current exercise, we have set this to abs(slope) in (0.41...0.88). This is fine for steady driving on a highway, but it also means that the car is not going to be able to detect lanes to which it is perpendicular (slope=0) such as when making a turn at an intersection. We are also going to sometimes detect other car outlines and various objects as lanes, although history helps a little bit with this.

#### Minor shortcomings
1. Currently all history is saved, not only the last n steps. The growing history list can introduce problems when algorithm is running on low-memory devices for long time. 
2. There is quite a lot of heuristic parametrization in the process. Not only for preprocessing, but also for clipping the slopes, defining RoI etc. We observed from challenge.mp4 that if small details such as the camera position, road color or lane markings change, it can throw the algorithm haywire.
3. The past 15 frames should not have equal vote on the slopes and intercepts, because the very last ones are more relevant.
4. The camera position can hinder current RoI performance. Already in challenge.mp4, the nose of the car became a redundant part of our RoI, throwing out zero slope lines.

### 3. Suggest possible improvements to your pipeline

#### Improvements for major shortcomings
1. Detecting lanes in arbitrary positions is a complex task. One way to make the algorithm generalize better is to introduce many additional if-then rules that try to account for different scenarios, such as position of the car, movement and consistency of slopes (to filter out car outlines), using sensor fusion to integrate additional information into making the lane decisions etc. However, this rule system grows massive very fast and in the end will never generalize as well as a human decision maker would. A more promising approach is deep learning, where we can implicitly learn all the necessary inferential rules by simply feeding the computer hours of driving data. 

#### Improvements for minor shortcomings
1. We can continuously throw away most of the history and keep in memory only what is relevant for the current situation.
2. This is essentially the same issue as in point 1. Deep learning will also help to prevent excess parametrization of if-then rules (even though neural networks themselves will need a lot of parametrization).
3. We can use exponential decay weights when doing the averaging over frames.
4. We can introduce an additional parameter  for our trapezoid which can raise it higher.
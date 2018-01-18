## Project: Search and Sample Return
---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[rock_images]: ./misc/rock_test_images.JPG
[rock_images_thresh]: ./misc/rock_test_thresh.JPG
[mask_image]: ./misc/mask_image.JPG
[obstacle_thresh]: ./misc/obstacle_thresh.JPG
[forward]: ./misc/forwardmode.png
[rock]: ./misc/I-FoundRock.png
[pickup]: ./misc/picking-up.png
[turning]: ./misc/turning.png

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

##### rock_samples
	  - I worked with an interactive matplotlib window to find a range of RGB values for yellow rocks. 
	  - I found that values for R are normally higher than 130. Values for G are higuer than 110 and values for B are less than 50.
	  - I tested all the images below.
![alt text][rock_images]
      - I defined rock_thresh and I could thresholding yellow rocks.
 ```python
      def rock_thresh(img, rgb_thresh=(130, 105, 50)):
    	color_select = np.zeros_like(img[:,:,0])
    	above_thresh = (img[:,:,0] > rgb_thresh[0]) \
                	& (img[:,:,1] > rgb_thresh[1]) \
                	& (img[:,:,2] < rgb_thresh[2])
    	color_select[above_thresh] = 1
    	return color_select
 ```
![alt text][rock_images_thresh]

##### obstacles
	  - For the obstacles, first I define a mask in perspective_transform function.
	  - I combine this mask with ground thresholing to obtain the obstacles thresholding.
	  - To create a mask, I get an array of ones from img[:,:,0] and then I apply perspective transform.

 ```python
mask=cv2.warpPerspective(np.ones_like(img[:,:,0]),M,(img.shape[1],img.shape[0]))
 ```
![alt text][mask_image]

	  - Then, I apply the following code to get obstacle thresholding.

 ```python
warped_obs, mask_obs = perspect_transform(obs_img, source, destination) 
threshed_obs=color_thresh(warped_obs)
obs_map=np.absolute(np.float32(threshed_obs)-1)*mask
```
![alt text][obstacle_thresh]



#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
And another! 

	- Call the `perspective_transform()` function to get a sky view (warped) and get a mask from this view using the same perspective
	transform technique to  a single layer image of ones. 
	- Get navigable terrain through `color_thresh()`function. 
	- Get obstacle terraing through the mask obtained from the previous step.
	- Get rover coordinates of transformed navigable terraing and obstacle terrain . 
	- Convert this rover coordinates to world_map coordinates.
	- Update `data.worldmap` with navigable terrain (blue layer) and obstacle terrain (red layer) world pixels with one layer for each one. If there is navigable terrain (`pixel values > 1`), I set obstacle terrain to `0`.
	- Find yellow rocks with `rock_thresh()`function. If there are rocks, update `data.worldmap[]` in the green layer.

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

##### Perception Step
	- The `perception_step()` is the same as the jupyter notebook functions. When the rover find a rock, I update the rover mode to `I found a rock` mode.

##### Decision Step
	- Here, I defined  the conditions for three rover modes : `I found a rock`, `Forward`, `Stop`
	- If the rover mode is `I found a rock`, it will reduce the throttle to 0.2 or 0 (depends of the current velocity), set rover.brake to 0 and set rover.steer with the correct angles. So, when the rover is near to a rock, it breaks and pick up the rock. Then, the rover mode is update to `Stop`.
![alt text][rock]
	- If the rover mode is `Forward`, the rover evaluate two possibilities. If rover.nav_angles is greater than 50 (if there is navigable terrain), the rover goes forward. If there is not navigable terrain, the rover breaks and goes to `Stop`mode.
![alt text][forward]

![alt text][pickup]
rover mode is `Stop`, it just breaks. Then, evaluate if there is navigable terrain (`nav.angles > 500`) it goes forward, if there is not navigable terrain (`nav.angles > 500`), it turns 15 degrees.
![alt text][turning]

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  
	- The rover maps 40% of the environment with more than 60% of fidelity.
	- The rover locate and pick up more than one rock. However, it takes too much time to pick up all the rocks and fidelity is affected.
	- Fidelity could be improved working with a range of roll and pitch, because  terrain is not flat and perception is affected.
	- There are situations when the rover turns in circles for a long time.

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  





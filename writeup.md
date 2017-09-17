# **Lane Lines Detection on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Create a pipeline that finds lane lines on the road from a video
   
* Reflect on the work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/intermediate_output/whiteCarLaneSwitch_grayscale.jpeg "Grayscale"

[image2]: ./test_images_output/intermediate_output/whiteCarLaneSwitch_CannyEdge_ROI.jpeg "Grayscale"

[image3]: ./test_images_output/whiteCarLaneSwitch.jpeg

---

### Reflection

### 1. Pipeline Details

My pipeline consisted of 7 steps. Following are the details of each of the steps

1. I have converted the images to grayscale.
2. Output of grayscale is passed through gaussian_blur() with kernel size set to '5' to smoothen out the image. 
3. Output from above step is passed to canny() function to detect the edges, low_threshold and high_threshold are set to 100 and 200.
4. Output from step 3 is passed to region_of_interest() function along with 'dim' list. dim list had to be changed depending upon the video being analyzed as the resolution and frame size was changing depending upon the video. All the test images and test vidoes had the same dimensions except for 'challenge.mp4' video. region_of_interest() function blackens out all the image except the region within 'dim' list area.
5. Output from step 4 is passed through cv2.HoughLinesP() , here i did not use the helper function as i wanted intermediate output. rho = 1,theta =1 deg,max_line_gap =200 and threshold , min_line_len had to be changed based on which image/ video being analyzed values like 15 /50 were used.
6. Output of step 5 are passed to draw_lines() function, this function was modified to accept 6 arguments namely 
   Arguments: 
    - img = grayscale image with only region of interest image
    - lines = HoughLinesP output
    - dim = region of interest coordinates
    - avg_len = decides the number frames data to be considered for averaging
    - color = color of the line
    - thickness = thickness of the line
   
   Description: The draw_lines function does the following operations.
    - Idenfity Right/ Left Lane: The HoughLinesP function output are passed to function identify_left_right_lines(), this function shall segregate Right and Left lines based on the sign of the lines slope. Line with +ve Slope is a Right Line and Line with -ve Slope is Left Line. Besides this the function shall also remove the horizontal lines. The segregate lines are returned by the function.
    - The output of identify_left_right_lines is 2 lists (right_xy,left_xy). These cannot be plotted on the image directly as there many individual small lines. To determine a common line the function draw_lines() removes deviations, averages and plots following are the details.
    
        __**Remove Deviation:**__
        - Remove all the lines that deviate a lot from the common trend , function reject_outliers() does this step
        
        __**Average:**__
        - Lanes were identified from the cleaned up lists. i.e. starting and ending points were identified from the cleaned up list. Left Lane: Starting Point (min(x),max(y)) , Ending Point: (max(x), min(y)). Refer to code for Right Lane.
        - If the lanes are directly plot then alot of oscillations were observed so an average filter was used to aide this buffers for Right and Left lanes were created to store lanes identified in last 300 frames. Functions fill_right_lane_buf() and fill_left_lane_buf() implement this.
        - find_line_param() function implements the average filter ,it shall use the stored  sample and generate 2 coordinates per each lane. From these coordinates slope (y2-y1/x2-x1) and constant (y-mx) are calculated. 
        
        __**Draw Lines:**__
        - using the slope and constant x coordinates for starting and ending points are calculated with region of interest as y- coordinates. finally draw the line in desired color.

Following  images show the intermediate and final outputs of whiteCarLaneSwitch.jpeg file. Refer the test_images_output folder to check results of all images
-__**Grascale Image**__
![GrayScale Image][image1] 
-__**Canny Edge Detection**__
![CannyEdge Detection][image2]
-__**Image with Lanes highlighted**__
![Image with Lanes][image3]
  

### 2. As per my understanding following are the potential shortcomings of this method
- This implementation cannot properly handle road curves
- In my implementation sometimes the overlayed lane deviates from the actual one 
- It takes 3 ~ 4 times of the actual video time to prefrom the operations  , i.e a 10sec video takes 30~40 sec to process. On vehicle such delay may not be acceptable.

### 3. Possible improvements:
- Instead of average filter a different filter with better performance needs to be implemented.
- In my current implementation i am trying to fit straight lines , instead curves should be used.
With the above modifications a curved road could be identified and processing time could be reduced.

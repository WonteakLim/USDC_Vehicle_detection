##Udacity-AD Project 5
---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # "Image References"
[HOG Feature]: ./figures/hog.png
[Detection]: ./figures/detection_result.png
[Result]: ./result_vehicle_detection.mp4

####  

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

HOG feature extraction exists on `extract_features()` in `functions.py`

I used `skimage.feature.hog` to get the hog feature and hog image. This function has three parameters: orientation, pixels_per_cell, and cells_per_block.

Here is an example using the `R Channel` of `RGB` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image1]



####2. Explain how you settled on your final choice of HOG parameters.

In this project, I used three types of features: HOG, spatial feature, and color histogram. All feature are described in detail below.

| Features        | Channel    | Option    | Function             |
| --------------- | ---------- | --------- | -------------------- |
| HOG             | 3 channels |           | `get_hog_features()` |
| Spatial feature | YCrCB      |           | `bin_spatial()`      |
| Color           | YCrCB      | Histogram | `color_hist()`       |

These features were concentrated on a serial vector. For feature balance, the vector was normalized by `StandardScaler()`

Through various tests, I optimized the parameters for feature extraction. 

All parameters were chosen below.

>   Color_space = 'YCrCb'
>
>   Spatial_size = (32,32)
>
>   orient = 9
>
>   pixel_per_cell = 8
>
>   cell_per_block = 2
>
>   hog_channel = 'ALL'
>
>   spatial_feat = True
>
>   hist_feat = True
>
>   hog_feat = True



Finally, feature vector length is `8460`.



####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

For training, I used the labeled data for vehicle and non-vehicle. This data were provided by the GTI vehicle image database, the KITTI vision benchmark suite, and the Udacity labeled dataset. Based on the data, the two types (vehicle and non-vehicle) of data can be divided by SVM classifier. The total features are `8460`.

In `Project5_training.ipynb`, there are training data import, parameter setting, feature extraction, randomized training data, SVM training, and model save. In particular, randomized training data divided the total dataset into two group for training and validation.



###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

In `find_cars()`, scale is a very important factor to detect vehicle with variant distances. At `scale=2.5`, this algorithm can detect closer vehicle (black), but not detect far vehicle (white) as shown below. For long range detection based on this method, I used multiscale and multisize windows.



The closer vehicles is to ego, the bigger. Using this fact, I selected three types of windows in `find_cars()`.

| Scale | y_start | y_stop |
| ----- | ------- | ------ |
| 1.0   | 380     | 500    |
| 1.4   | 400     | 600    |
| 2.5   | 500     | 700    |



####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

These are some results of test images for detection vehicles. The blue boxes are the positive windows of vehicle detection. These has many false positives. To reject this, I used heatmap described in the Udacity lecture. The map represented the count of pixels integrated by the blue boxes. By thresholding method, these pixels can be filtered as shown below. (the upper left image)

![alt text][image4]
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video 
Here's a [link to my video result](./result_vehicle_detection.mp4)




####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

For filtering false positive, I recorded the heatmaps in the last three fames of the video. These heatmaps of sequential frames are integrated one heatmap in each frame. I thresholded that map to find positive detection of vehicles. Actually, these method had pros&cons. I could reject many false positives by long frame integration. However, it occurs a delay of detection. For this reason. the number of frames for integration should be selected carefully.

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In this project, I used the average threshold method for robust detection. This algorithm is very simple and strong for this video. However, this algorithm could not divided overlapped vehicles. This problem could not be solved by image processing, only.

In the future, I will apply T2T(Track-to_Track) fusion algorithm and a kalman filter tracking based on the vehicle's kinematic model. Adding these method into the current algorithm, I expected that the performance of vehicle tracking is better.
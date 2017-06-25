### Jianguo Zhang, June 24, 2017
## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/dataset_visualize.jpg
[image2]: ./output_images/dataset_hog_example.jpg
[image3]: ./output_images/sliding_window_example.jpg
[image3_1]: ./output_images/advanced_sliding_window_example.jpg
[image4]: ./output_images/sliding_window_example_test5.jpg
[image5]: ./output_images/heat_map_no_thresh_test5.jpg
[image5_1]: ./output_images/heat_map_with_thresh_test5.jpg
[image5_2]: ./output_images/heat_map_with_thresh_examples.jpg
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1 . Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code `cell 2-6`. 

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.


Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

####2. Explain how you settled on your final choice of HOG parameters.

The code for this part can be found in code `cell 7-8`.

I tried various combinations of parameters. The parameters I use shows as following:

 |feature vector length X_train:  6156 |Scaled_X_train:  6156 |
 |Color Space:  YCrCb |orient:  9 |
 |pixels per cell:  8 |cells per block:  2 |
 |hog_channel:  ALL |spatial_size:  (16, 16) |
 |hist_bins:  32 |ystart:  400 |
 |ystop:  656| number of frames: 10|
 |scale: 1.5|xy_overlap: (0.5, 0.5)|

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The code for this part is in code `cell 9-11`.

First, I normalize the combined features, then I trained a linear SVM using LinearSvc, the dataset is randomly split into training and testing set, where the test_size is 0.2. 

I also try the automatically parameter tuning using GridSearchCV with parameter list  {'kernel':('linear', 'rbf'), 'C':[1, 10]}. Howerver, It takes long time to train although it can get the best parameters.


### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The code for this part contained in code `cell 12-19`.

This part code is mainly taken from lesson material,the overlap is set to (0.5, 0.5):


![alt text][image3]

You can see that for this method there are some false positive windows and some cars and not detected. we now try some efficient method. The mehod can both extract features and make predictions. It only has to extract hog features once and then can be sub-sampled to get all of its overlaying windows.

The code for this part can be found in code `cell 20-22`.

One example like this:

![alt text][image3_1]

#### 2. False positive and Heat mapping

The code for this part contained in code `cell 24-29`.

You can see that for this method there are some false positive windows and some cars and not detected. like the following example:

![alt text][image4]

We convert the "hot windows" tofind the heat map,  the "hot" parts of the map are where the cars are, and using a threshold to find the true positive cars.

Before using the threshold:

![alt text][image5]

After applying threshold, by imposing a threshold, it can reject areas affected by false positives:

![alt text][image5_1]


#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image5_2]
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_result.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The code for this part can be found in code `cell 30-last`. 

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

I define a `Detection` class to smooth several frames, here I choose 10 frames. Every time I find the heat map of 10 frames, it can be helpful to reject false positive in orderinf list of images. It can be found in the `process_each_frame function` of code 'cell 34'. 


---

###Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

It is hard to detect strong false positive examples. I did not combine multiple scaling method, this may also make the pipline not so robust.

In future, I will try some more advanced real-time method like  [Faster R-CNN](https://arxiv.org/pdf/1506.01497.pdf) in 2016 and the newest method [Mask R-CNN](https://arxiv.org/pdf/1703.06870.pdf) in 2017.

# CarND-Vehicle-Detection-P5


**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector.
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: /output_images/samples1.png
[image2]: /output_images/hog_feat_car.png
[image3]: /output_images/hog_feat_noncar.png
[image4]: /output_images/hog_pix_cell.png
[image5]: /output_images/hog_ycrcb.png
[image6]: /output_images/hog_heatmap.png
[image7]: /output_images/labels1.png
[image8]: /output_images/detected1.png
[image9]: /output_images/detected2.png
[image10]: /output_images/detected3.png
[image11]: /output_images/detected4.png
[image12]: /output_images/detected5.png
[image13]: /output_images/detected6.png
[image14]: /output_images/heat1.png
[image15]: /output_images/heat2.png
[image16]: /output_images/heat3.png
[image17]: /output_images/heat4.png
[image18]: /output_images/heat5.png
[image19]: /output_images/heat6.png
[image20]: /output_images/final_labels.png
[image21]: /output_images/final_detect.png
[image22]: output.gif

[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.


I started by reading the filenames and creating list for the `vehicle` and `non-vehicle` images. There are 8792 `car` images and 8968 `non-car` images in the dataset.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I used the sample images to visualize the difference is between a car and non-car image when extracting HOG features:

![HOG Features Car][image2]

![HOG Features Non-Car][image3]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`). I tested how increasing the pixels_per_cell parameter affected the HOG output:

![HOG Pix per Cell][image4]

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)` and one channel:

![alt text][image5]

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters, mainly evaluating different color
spaces and channels. I defaulted to using pix_per_cell = 8, cell_per_block=2, and orient=9 to compare the performance of using channels and color spaces.

Using RGB and one channel yielded a 93.9% accuracy on the test set. YCrCb color space and one channel yielded 95.2% accuracy. Using all channels and still using YCrCb yielded an accuracy of 98.9%. I decided to use YCrCb color space and the three channels for the final implementation.



#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I tried both color and HOG classification independently to experiment on the performance based on this features. Color classification on its own achieved  an accuracy of 93.8%. HOG classified achieved an accuracy of 98.7% . I then experimented combining color and HOG features to test that additional features would indeed increase the performance.

Based on testing different color spaces and channels I decided to use YCrCb color space and 3-channel HOG features. I applied spatial binning and histograms of color to obtain more features. The result was a feature vector with all three sets of features concatenated that can be fed to a Linear SVC. The final accuracy on the test set was 99.4%. To compare the performance and impact of using a different color space, I trained the model using RGB color space and achieved an accuracy of 97.9%, which suggest that the final combination of color space and channels was a better alternative.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I tried using the sliding window method but ended up using the HOG subsampling method as it applies HOG function to the whole image reducing latency. The method `find_cars` applies the HOG function to the entire image and then subsamples multiple windows based on a specified start and stop position, and a scale. For each window, a feature vector is created that includes the HOG features for three channels and also features form spatial binning and histograms of color.

The feature vector for each of the sampled regions can then be scaled and fed into the Linear SVC to generate a prediction. Here is an example of the predicted windows using this method and the corresponding heatmap:

![alt text][image6]



#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

To optimize the performance of the classifier, the combined feature vector was used to train the classifier as it yielded the best accuracy. As detailed in the previous section, the combination of HOG features, histograms of color, and spatial binning produced true detections and a small number of false positives. Hard negative mining might improve the performance but I did not use it for the final implementation.

 Here are example detections on test images:


![sample detection][image7]
![sample detection][image8]
![sample detection][image9]
![sample detection][image10]
![sample detection][image11]
![sample detection][image12]
![sample detection][image13]

---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

My final pipeline was the following:
1. Find detections at 1.25, 1.5, and 1.75 scale.
2. Generate a heatmap from detections and add it to a list of the last 10 frames.
3. Apply a threshold to the combined heatmap.
4. Generate labels from the heatmap.
5. Draw the bounding boxes on the frame.


Here's a [link to my video result](./project_video.mp4)

![gif][image22]


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

In the final pipeline, additional window scales are used to increase the detection performance and be able to recognize vehicles with different scales.I stored detections for 10 consecutive frames and then applied a threshold to the resulting heatmap. The reason behind this is that over several frames positive detections will get 'hotter' and when the threshold is applied, only 'cold' pixels will be removed. This assumes that over several frames there will not be an increasing number of false positives at the same location.

The thresholded heatmap can then be used to detect different vehicles on the frame using `scipy.ndimage.measurements.label()` method. This provides the the number of vehicles detected and a corresponding labels that can be used to generate bounding boxes on the vehicles.


### Here are six frames and their corresponding heatmaps:

![heat1][image14]
![heat2][image15]
![heat3][image16]
![heat4][image17]
![heat5][image18]
![heat6][image19]


### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![final labels][image20]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![final detect][image21]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The final implementation work reasonably well at detecting vehicles on the video. One of the challenges I encountered was eliminating all false positives while adding different scales to detect vehicles at different distances. In other words, when decreasing the scale of the detection windows, there where an increased number of false positives but when filtering them vehicles further in the horizon would appear as not detected. With further experimentation, these parameters can be better optimized to achieve a more robust pipeline.

The second challenge is the time for processing frames. One downfall is that is very time consuming to experiment with the entire video and optimize the parameters. Depending on the scale of windows for detection could take close to 1.5s, which also means that the system cannot work in real-time and therefore cannot be used in a real scenario.
